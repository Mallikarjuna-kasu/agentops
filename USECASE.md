# Use Case: Multi-Team LLM Cost Governance

## Scenario

A mid-size enterprise runs six internal teams on a shared LLM infrastructure — Engineering, Data Science, Operations, Marketing, Executive Strategy, and Customer Support. Each team has a monthly token budget. No one monitors individual calls.

**What happens without AgentOps:**

The Engineering team submits a 120M-token document analysis batch on a Friday afternoon. The single API call costs \$4,500. A second batch on the same day adds \$2,200. By Monday, \$6,067 is gone from a \$2,000 monthly budget — 303% over. No alert fired. No one was asked. The bill arrives in 30 days.

Meanwhile, the Marketing team runs an unthrottled campaign automation job — mistral-large-3 at 280M tokens — adding \$1,320 against a \$300 budget (455% over). Finance sees a combined \$10,611 bill with no attribution, no audit trail, and no idea which calls were legitimate.

**This is the problem AgentOps solves.**

---

## Workflow with AgentOps

### Step 1 — User submits request
User selects their team and submits a prompt via the dashboard. The system records: team ID, estimated token count, model selection.

### Step 2 — Budget check
Backend queries real-time team spend:
> *"Engineering remaining: \$2,000 monthly budget. Current spend: \$1,240. This request estimated at \$4,500 — over budget by \$3,740."*

### Step 3 — Approval routing
Over-budget request is flagged `PENDING_APPROVAL`. Slack and/or email alert dispatched to the team's approver.

### Step 4 — Human decision
Approver reviews the request in the Approvals queue: risk level, estimated tokens, cost, and expected impact. One click to Approve or Reject. Decision is logged with identity and timestamp.

### Step 5 — Execution (if approved)
LangGraph agent workflow launches: Research → Critic → Synthesis nodes. Every node logs its duration, token count, and cost to the trace store.

### Step 6 — Anomaly detection runs
Z-score engine evaluates every completed event. Calls at 3σ+ above baseline are classified as critical anomalies. Recommendations are generated automatically.

### Step 7 — Dashboard updates
Team burn-down, cost trend, provider breakdown, and anomaly feed all update in real time. Team leads see their budget status without waiting for the monthly invoice.

---

## Actors

| Role | Responsibility |
|---|---|
| **Requester** | Submits prompts, selects team, sees cost estimates |
| **Approver** | Receives over-budget alerts, approves or rejects agent execution |
| **Admin** | Sets team budgets, reviews cross-team analytics, monitors anomalies |

---

## Observed Outcomes (Demo Data)

The platform is seeded with 44 real enterprise-scale events across 13 users, 10 LLM models, and 4 active teams.

| Outcome | Value |
|---|---|
| Total spend detected | \$10,611.88 |
| Anomalies caught | 16 (12 critical) |
| Single largest call flagged | gpt-5.5 · \$4,500 · 5.8σ |
| Teams over budget | 4 of 6 |
| Budget overspend (Engineering) | 303% |
| Budget overspend (Marketing) | 455% |
| Avg latency across all calls | 1,218ms |
| Error rate | 6.82% |

Without AgentOps: all of the above is invisible until the invoice.
With AgentOps: every event is attributed, every spike is flagged, every agent action requires a human decision.
