# Use Case: Multi-Team LLM Cost Governance

## Scenario
A mid-size enterprise has three internal teams using a shared LLM API:
- **Data Science Team** — Heavy batch analysis jobs
- **Product Team** — Ad-hoc research queries
- **Finance Team** — Occasional report generation

## Risk
The Data Science team submits a 10,000-row dataset for analysis at 4:00 PM Friday. The job consumes $340 in tokens before Monday. No one is alerted. The bill is unassigned.

## Workflow with AgentOps

1. **User submits request** via the dashboard with team selection.
2. **System checks budget** — "Data Science remaining: $120 / $500 daily."
3. **Cost estimate** — "This request is estimated at $45. Proceed?"
4. **Execution** — If under budget, the multi-agent workflow launches.
5. **Trace capture** — Every agent node, latency, and token count is recorded.
6. **Dashboard update** — Real-time burn-down chart updates for the team lead.

## Actors
- **Requester** — Submits prompts, sees cost estimates.
- **Approver** — Receives over-budget alerts, approves/denies.
- **Admin** — Sets budgets, reviews cross-team analytics.
