---
title: "Event 3"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “Agent Forge - Deepdive Day 3”

### 1. Event Overview
- **Date & Time**: 9:00 AM – 12:00 PM, Saturday, August 15, 2026
- **Location**: 26th Floor, Bitexco Financial Tower, 2 Hai Trieu St., Ho Chi Minh City
- **Role**: Attendee

### 2. Speakers
- **Nghia Tran** — Agentic SA
- **Anh Pham** — Cloud Consultant, G-AsiaPacific Vietnam

---

### 3. What Was Covered

#### Theory

##### Memory
Without memory, an agent forgets previous turns once the context window fills up. The session covered how to keep useful context across a conversation:

- **Short-term memory**: recent chat history, easy to pull back when needed
- **Long-term memory**: insights extracted from past chats, stored as vectors
- **Strategies**: Summary, User Preference, Semantic, Episodic
- **Namespace**: paths like `/Strategy/Actor/Session` to scope search, cut token use, and speed up retrieval

##### Evaluations
Before shipping an agent, you need a way to check if answers are correct, useful, and safe — and catch hallucinations or bad tool calls.

- **On-demand**: run checks while building
- **Online**: keep monitoring in production with telemetry

Checks can run at three levels:
- **Session** — whole conversation
- **Trace** — a single reply
- **Span** — a tool call and its parameters

A **Judge** reviews agent activity; results go into Observability so the team can step in when something looks off.

##### Observability
Basically: know what the agent did, how it did it, and what it cost.

- **Logs** — what happened
- **Traces** — the full path of a request
- **Metrics** — latency, token cost, error rate

Also covered OpenTelemetry, alerts, and the hierarchy `Session → Trace → Span`.

##### AgentCore building blocks
- **Registry** — share and reuse skills, tools, APIs (Admin / Publisher / Consumer)
- **Harness** — thin wrapper to spin up an agent from Model + System Prompt + Tools
- **Tools** — let the agent call external systems and live APIs
- **Payments** — agent-triggered payments (Stripe, Coinbase)
- **Optimization** — use eval/observability data for A/B tests, red teaming, and improvement loops
- **Policy** — guardrails: human-in-the-loop, Cedar, strict/permissive modes, least privilege

#### Hands-on
Walkthrough with Agent SDK, AWS Bedrock setup, and CLI: create a project, deploy, and test an agent on AWS.

---

### 4. Takeaways

Day 2 made it clearer how Memory, Evaluations, and Observability fit together when you actually run agents in production — not just demo them. AgentCore pieces (Registry, Harness, Tools, Policy, Optimization) are less abstract once you see how they connect for reuse, control, and iteration. Least privilege and human-in-the-loop stood out as practical safety defaults, not theory. The lab also helped me get comfortable with Agent SDK + Bedrock + CLI for the basic create → deploy → test flow.
