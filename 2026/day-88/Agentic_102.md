# Multi-Tool Agents, MCP, and CI/CD Analyzer

## Task 1: Build the Multi-Tool DevOps Agent (Module 3)

**Set up a Kind cluster with a broken pod:**

```
kind create cluster --name devops-demo
kubectl apply -f module-3/broken_pod.yaml

The broken_pod.yaml deploys a pod that crashes immediately:

apiVersion: v1
kind: Pod
metadata:
  name: broken-pod
  namespace: default
spec:
  containers:
  - name: app
    image: nginx:alpine
    command: ["sh", "-c", "echo 'app starting...' && sleep 2 && exit 1"]
```

**Also create a broken Docker container:**

`docker run -d --name broken-container nginx:alpine sh -c "echo 'container starting...' && sleep 2 && exit 1"`

**Study module-3/agent.py -- it has 6 tools now:**

Kubernetes tools (new):

```
@tool
def list_pods(namespace: str = "default") -> str:
    """List all pods in a Kubernetes namespace with their status."""
    result = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace],
        capture_output=True, text=True,
    )
    return result.stdout or result.stderr

@tool
def describe_pod(pod_name: str, namespace: str = "default") -> str:
    """Get detailed info about a Kubernetes pod including events and conditions."""
    result = subprocess.run(
        ["kubectl", "describe", "pod", pod_name, "-n", namespace],
        capture_output=True, text=True,
    )
    return result.stdout or result.stderr

@tool
def get_events(namespace: str = "default") -> str:
    """Get recent Kubernetes events in a namespace (useful for troubleshooting)."""
    result = subprocess.run(
        ["kubectl", "get", "events", "-n", namespace, "--sort-by=.lastTimestamp"],
        capture_output=True, text=True,
    )
    return result.stdout or result.stderr

```

**Run it:**

`python3 module-3/agent.py`

**Ask questions that span both domains:**
```
> What's broken across Docker and Kubernetes?
> Why is broken-pod crashing?
> Are there any unhealthy containers on Docker?
> Describe the events in the default namespace
The agent decides which tools to use based on the question. Ask about Docker -- it uses Docker tools. Ask about pods -- it switches to Kubernetes tools. Ask about both -- it uses all of them.

This is the power of the ReAct pattern: One agent, many tools, one brain that decides what to use.
```


## Task 2: Understand the Model Context Protocol (MCP)

The Model Context Protocol (MCP) is an open standard introduced by Anthropic in November 2024 to securely connect AI models and assistants to external data sources, tools, and workflows. Think of it as a universal "USB-C port" for artificial intelligence, replacing fragmented, custom API integrations with a single, standardized interface

**Why MCP matters for DevOps:**

| Without MCP | With MCP |
|------------|---------|
| Tools are locked to one framework (LangChain) | Tools work with any MCP client |
| Every AI client re-implements Docker/K8s tools | Write once, use everywhere |
| Tool access tied to the agent code | Tools exposed as a discoverable service |


**MCP-compatible clients:**

* Claude Desktop
* VS Code (GitHub Copilot)
* Cursor
* Claude Code (the CLI you might already be using)
* Any LangChain agent via langchain-mcp-adapters

**The architecture:**

```
[MCP Server]                    [MCP Clients]
  |                                  |
  |-- list_pods()                    |-- Claude Desktop
  |-- describe_pod()      <--->      |-- VS Code Copilot
  |-- get_events()                   |-- Your Python agent
  |                                  |-- Any MCP client
  |
  (exposes tools via stdio/HTTP)

```

## Task 3: Build and Use the MCP Server (Module 3)

Key difference from LangChain tools:

@mcp.tool instead of @tool -- registered with the MCP server
FastMCP("Kubernetes Tools") -- creates a named MCP server
mcp.run() -- starts the server (stdio transport by default)
Any MCP client can discover and call these tools


module-3/mcp_server.py, module-3/agent_with_mcp.py

The agent does not define tools locally. It connects to the MCP server and discovers them at runtime.

**Run the MCP agent:**
```
cd module-3
python3 agent_with_mcp.py
```

Same result as before, but the tools are served via MCP instead of being hardcoded in the agent.

Configure Claude Desktop with your MCP server (if you have Claude Desktop installed):

Add to ~/Library/Application Support/Claude/claude_desktop_config.json (macOS):

```
{
  "mcpServers": {
    "kubernetes-tools": {
      "command": "python3",
      "args": ["/full/path/to/agentic-ai-for-devops/module-3/mcp_server.py"]
    }
  }
}
```

Restart Claude Desktop. Now you can ask Claude: "List the pods in my cluster" and it will call your MCP server's list_pods() tool.

## Task 4: Build the CI/CD Failure Analyzer (Module 6)

The same agent pattern works for CI/CD. This agent uses the gh CLI to diagnose GitHub Actions failures.

module-6/ci_analyzer.py

Note the log truncation in get_failed_logs -- LLMs have token limits. You cannot send 100KB of CI logs. Truncating to 5000 characters keeps it within bounds while preserving the most important information (the failed step output).

**Run it inside the AI-BankApp repo (which has GitHub Actions):**

```
cd AI-BankApp-DevOps
python3 ../agentic-ai-for-devops/module-6/ci_analyzer.py
Ask:

> What failed in my last CI run?
> Show me the recent workflow runs
> Read the gitops-ci.yml workflow file and explain what it does
The agent lists failed runs, fetches their logs, reads the workflow file, and explains the root cause.
```

**Try creating a deliberately broken workflow to test it:**

Push it, let it fail, then ask the agent: "Why did broken-ci fail?"

## Task 5: Build Your Own Tool


The pattern is now clear. Any CLI command can be a tool. Build one of these:

Option A -- Terraform Plan Analyzer:

```
@tool
def terraform_plan() -> str:
    """Run terraform plan and return the output showing what would change."""
    result = subprocess.run(
        ["terraform", "plan", "-no-color"],
        capture_output=True, text=True,
        cwd="/path/to/your/terraform/project"
    )
    output = result.stdout + result.stderr
    if len(output) > 5000:
        output = output[:5000] + "\n[...truncated]"
    return output

```

**Option B -- AWS Resource Checker:**
```

@tool
def list_ec2_instances() -> str:
    """List all EC2 instances with their state, type, and name."""
    result = subprocess.run(
        ["aws", "ec2", "describe-instances",
         "--query", "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,Tags[?Key=='Name'].Value|[0]]",
         "--output", "table"],
        capture_output=True, text=True,
    )
    return result.stdout or result.stderr

```

**Option C -- Log Searcher:**
```
@tool
def search_logs(keyword: str, namespace: str = "default") -> str:
    """Search for a keyword in the logs of all pods in a namespace."""
    pods = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace, "-o", "name"],
        capture_output=True, text=True,
    )
    results = []
    for pod in pods.stdout.strip().split("\n"):
        if not pod:
            continue
        logs = subprocess.run(
            ["kubectl", "logs", pod, "-n", namespace, "--tail=100"],
            capture_output=True, text=True,
        )
        if keyword.lower() in logs.stdout.lower():
            results.append(f"{pod}: found '{keyword}'")
    return "\n".join(results) if results else f"No pods contain '{keyword}' in their logs"

```

Add your tool to any agent, run it, and ask a question that triggers it.

**The pattern is always the same:**

Define tools that wrap CLI commands
Create an LLM instance
Create a ReAct agent
The agent reasons about the question, calls tools, reads output, answers
