# AgentOps

**Enterprise LLM Cost Governance Prototype**

Multi-team LLM cost governance with approval-gated agent execution, real-time budget tracking, and full execution traceability.

---

## The Problem

Enterprise teams share LLM API keys without spend controls.
- Team A runs a $5 analysis.
- Team B runs 500 of them overnight.
- Finance sees the bill Monday morning with zero attribution.

No guardrails. No accountability.

---

## The Solution

| Feature | Outcome |
|---|---|
| **Per-Team Budgets** | Each team has a monthly cost ceiling. |
| **Approval Gates** | Over-budget or high-risk requests require explicit approval before execution. |
| **Execution Tracing** | Every agent step is logged with latency, tokens, and cost. |
| **Real-Time Dashboard** | Live spend tracking by team, model, provider, and time window. |
| **Anomaly Detection** | Z-score engine flags cost spikes, latency spikes, token abuse, and high error rates. |

---

## Architecture

```
User Browser
    |
    v
Dashboard UI (React + Vite + TypeScript)
    |
    v
Governance Backend (FastAPI + asyncpg)
    |                           |
    v                           v
Budget Check              State & Budget Store
    |                           |
    v                           |
Agent Orchestration <---------|
(LangGraph: Research -> Critic -> Synthesis)
    |
    +------> Trace & Audit Store (Langfuse)
    |
    +------> Checkpoint Cache (Redis)
    |
    v
Dashboard Update
```

**Data Flow:** Request -> Auth -> Budget Check -> Agent Execution -> Trace -> Dashboard Update

---

## Dashboard Views

### 1. Overview
Live system metrics and recent events.

| Metric | Value (Demo) |
|---|---|
| Total Requests | 201 |
| Total Cost | $7.6953 |
| Avg Latency | 1176ms |
| Error Rate | 6.97% |

Tracks cost trends over time and lists recent agent runs with model, team, tokens, latency, cost, and status.

![Overview](./assets/screenshots/01-overview.png)

---

### 2. Cost Analytics
Track LLM spend across models, teams, and time.

| Provider | Share |
|---|---|
| OpenAI | 49% |
| Anthropic | 27% |
| Mistral | 12% |
| Google | 6% |
| Meta | 4% |
| Alibaba | 2% |

Features cumulative cost trend, model cost comparison bar chart, and latency distribution.

![Cost Analytics](./assets/screenshots/02-cost-analytics.png)

---

### 3. Agent Traces
LangGraph execution history with step-by-step trace correlation.

| Status | Count |
|---|---|
| Completed | 2 |
| Pending | 1 |
| Failed | 1 |

Shows per-run duration, model, tokens, cost, and full execution log with node-level timing.

![Agent Traces](./assets/screenshots/03-agent-traces.png)

---

### 4. Approvals
Human-in-the-loop approval queue for agent execution.

| Status | Count |
|---|---|
| Pending Review | 2 |
| Approved | 4 |
| Rejected | 2 |

Pending requests show risk level (medium/low), estimated tokens, cost, and Approve/Reject actions. Approval history tracks all decisions with timestamp and approver.

![Approvals](./assets/screenshots/04-approvals.png)

---

### 5. Anomaly Detection
Z-score engine with threshold rules and real-time alerting.

| Metric | Value |
|---|---|
| Total Anomalies | 30 |
| Critical | 24 |
| High | 6 |
| Events Scanned | 201 |

Detects cost spikes, latency spikes, token abuse, and high error rates. Risk radar visualizes anomaly pressure by dimension.

![Anomaly Detection](./assets/screenshots/05-anomaly-detection.png)

---

### 6. Users
Top users by token consumption and cost.

| Rank | User | Team | Tokens | Cost | Calls |
|---|---|---|---|---|---|
| #1 | Kasu Mallikarjuna | Engineering | 772,770 | $3.7022 | 70 |
| #2 | Alex Kumar | Marketing | 247,180 | $0.2189 | 13 |
| #3 | Lisa Zhang | Marketing | 145,428 | $0.9345 | 13 |

![Users](./assets/screenshots/06-users.png)

---

### 7. Team Management
Configure teams, budgets, users, and access controls.

| Team | Monthly Budget | Spent This Month | Remaining | Status |
|---|---|---|---|---|
| Marketing | $300.00 | $0.1284 | $299.8716 | Healthy |
| Customer Support | $500.00 | $0.0000 | $500.0000 | Healthy |
| Executive Strategy | $1500.00 | $0.0000 | $1500.0000 | Healthy |
| Engineering | $2000.00 | $5.8530 | $1994.1470 | Healthy |

![Team Management](./assets/screenshots/07-team-management.png)

---

## Use Case and Workflow

See [USECASE.md](./USECASE.md) for the full business scenario.

## Inputs and Results

See [INPUTS-RESULTS.md](./INPUTS-RESULTS.md) for example inputs and system outputs.

---

**Built under the [PalyamIQ](https://palyamiq.com) technical portfolio.**
