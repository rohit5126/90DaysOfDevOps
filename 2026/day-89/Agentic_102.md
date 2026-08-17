# Production AI Agents: KubeHealer and AIOps

**Today we are gonna built a AI agent which solves the issue.**

#### Task 1: Understand AIOps and Production Guardrails (Module 4)

**what i AIOps**

AIOps stands for Artificial Intelligence for IT Operations. It uses big data, machine learning, and analytics to automate and improve IT management. It collects data from different tech systems, filters out false alarms, spots hidden patterns, and fixes problems

**Production guardrails every AI agent needs:**

| Guardrail | Why | Example |
|-----------|-----|---------|
| **Human approval** | Agents should not make destructive changes without permission | "I found 3 broken pods. Here are the fixes. Approve?" |
| **Scope limits** | Agents should only operate in allowed namespaces/clusters | Cannot touch `kube-system` or production databases |
| **Audit trail** | Every action must be recorded | Temporal workflow history: every tool call, every decision |
| **Rollback capability** | Every fix must be reversible | Agent creates patches, not replacements |
| **Timeout and retry limits** | Agents must not loop forever | Max 3 retries per pod, timeout after 5 minutes |
| **Escalation path** | When the agent cannot fix it, alert a human | "config-app needs a ConfigMap I cannot create. Escalating." |

**Why durable execution (Temporal) matters:**

Durable execution via platforms like Temporal matters because it guarantees that long-running application code runs to completion—surviving server crashes, network drops, and infrastructure outages without losing state or requiring manual recovery.

**When to use AI agents vs traditional automation:**

| Use AI Agents When | Use Traditional Automation When |
|--------------------|---------------------------------|
| Problem requires reasoning (diagnose unknown errors) | Problem has a known, fixed solution |
| Multiple possible causes and fixes | One cause, one fix (if X then Y) |
| Natural language output helps humans | No human in the loop |
| Examples: troubleshooting, root cause analysis | Examples: scaling, restarts, deploys |



