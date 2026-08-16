# Introduction to Agentic AI for DevOps

## Task 1: Understand Agentic AI for DevOps

#### What is an AI agent?

An AI agent is an autonomous software system powered by artificial intelligence that can perceive its environment, make multi-step plans, and use tools or APIs to accomplish specific goals without constant human supervisio

#### Why agents for DevOps?

DevOps agents are autonomous AI systems designed to observe, decide, and execute tasks across the software delivery lifecycle. They solve modern DevOps bottlenecks by handling repetitive incident triage, analyzing complex log data, enforcing compliance, and optimizing cloud costs without requiring manual scripting for every single operational hurdle

#### The ReAct pattern (Reason + Act):

```
User: "Why is broken-app crashing?"

Agent THINKS: I should check which containers are running
Agent ACTS:   calls list_containers()
Agent OBSERVES: broken-app is in "Restarting" state

Agent THINKS: I should check the logs
Agent ACTS:   calls get_logs("broken-app")
Agent OBSERVES: "exit code 1" after "app starting..."

Agent THINKS: The container exits immediately after starting
Agent ANSWERS: "The container crashes because the entrypoint
                command exits with code 1 after 2 seconds..."

```

## Task 2: Set Up the Environment

**clone the repo**

```
git clone https://github.com/rohit5126/agentic-ai-for-devops.git
```

**Install Ollama**

```
curl -fsSL https://ollama.com/install.sh | sh

```
**start ollama and pull free model**
```
ollama serve &
ollama pull gemma4
```

**Set up Python environment:**
```
python3 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
```

**run check**
```
python3 module-0/verify_setup.py
```


## Task 3: Build the Docker Error Explainer (Module 1)

Run it:
```
python3 module-1/explainer.py
Paste one of these Docker errors:

docker: Error response from daemon: Conflict. The container name "/myapp" is already in use.
Or:

Error response from daemon: driver failed programming external connectivity on endpoint myapp:
Bind for 0.0.0.0:8080 failed: port is already allocated.
Or:

Error response from daemon: pull access denied for mycompany/private-app, repository does not
exist or may require 'docker login'.

```
The LLM explains what went wrong and how to fix it -- no manual Googling needed.

A good system prompt makes answers precise and steady. A poor or vague prompt leads to random, messy, or unsafe replies.

## Task 4: Build the Docker Troubleshooter Agent (Module 2)

**create a broken container**

`docker run -d --name broken-app nginx:alpine sh -c "echo 'app starting...' && sleep 2 && exit 1"`

**The agent is created with:**
```
llm = ChatOllama(model="gemma4", temperature=0)
tools = [list_containers, get_logs, inspect_container]
agent = create_react_agent(llm, tools)
```

**Run the agent:**
```
python3 module-2/agent.py
Ask it:

> Why is broken-app crashing?
Watch the agent's reasoning:

It calls list_containers() -- sees broken-app in "Restarting" state
It calls get_logs("broken-app") -- sees "app starting..." then exit
It calls inspect_container("broken-app") -- sees exit code 1
It answers: "The container crashes because the command exits with code 1..."
The LLM decided which tools to call and in what order. You never told it to check logs -- it figured that out from the problem.
```

**Try more questions:**

> List all my running containers
> What image is broken-app using?
> Is any container using port 8080?

**Clean up:**

`docker rm -f broken-app`

## Task 5: Understand the Agent Architecture

Map out what you just built:
```
[User Question]
      |
      v
[LLM: Gemma 4 via Ollama]
      |
      | (ReAct: Reason what tool to use)
      v
[Tool Selection]
      |
      +---> list_containers()   --> docker ps -a
      +---> get_logs()          --> docker logs
      +---> inspect_container() --> docker inspect
      |
      v
[Tool Output (text)]
      |
      v
[LLM reads output, reasons again]
      |
      | (repeat until answer is ready)
      v
[Final Answer to User]

```

**Why this matters for DevOps:**

The pattern is domain-agnostic. Replace Docker tools with Kubernetes tools, Terraform tools, or AWS CLI tools -- the architecture stays the same
Tomorrow (Day 88) you will add Kubernetes tools to the same agent
On Day 89, you will build a production-grade agent that automatically fixes broken pods
The tool pattern is always the same:

```
@tool
def my_tool(argument: str) -> str:
    """Description the LLM reads to decide when to use this tool."""
    result = subprocess.run(["some-cli", "command", argument], capture_output=True, text=True)
    return result.stdout or result.stderr
```
Any CLI command can become an agent tool. Any DevOps workflow can be automated this way.

## Task 6: Experiment and Extend

Try adding a new tool to the agent. Edit module-2/agent.py and add:

```
@tool
def list_images() -> str:
    """List all Docker images on this machine with their sizes."""
    result = subprocess.run(["docker", "images"], capture_output=True, text=True)
    return result.stdout or result.stderr

```
**Add it to the tools list:**
```
tools = [list_containers, get_logs, inspect_container, list_images]
Run the agent and ask: "What images do I have and how much space are they using?"

The agent will call your new tool.
```
**Try another: Add a restart_container tool:**
```
@tool
def restart_container(container_name: str) -> str:
    """Restart a Docker container."""
    result = subprocess.run(["docker", "restart", container_name], capture_output=True, text=True)
    return result.stdout or result.stderr
```
**Now ask: "broken-app keeps crashing, can you restart it?"**

Think about the safety implications: This tool can restart any container. In production, you would add guardrails (confirmation prompts, allowed container lists). You will learn about guardrails on Day 89.

