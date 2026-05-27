# Architecture

## System Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  User Interface  │────▶│  Governance     │────▶│  State & Budget │
│  (Dashboard)     │     │  Backend        │     │  Store          │
│                  │◄────│                 │◄────│                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
           ┌─────────────────┐   ┌─────────────────┐
           │  Budget Guard   │   │  Anomaly Engine  │
           │  per team/call  │   │  Z-score on      │
           │                 │   │  cost/lat/tokens │
           └─────────────────┘   └─────────────────┘
                    │
                    ▼
           ┌─────────────────┐
           │  Agent          │
           │  Orchestration  │
           │  Engine         │
           │                 │
           │  • Research     │
           │  • Critic       │
           │  • Synthesis    │
           └─────────────────┘
                    │
           ┌────────┴────────┐
           │                 │
           ▼                 ▼
  ┌─────────────────┐  ┌─────────────────┐
  │  Trace & Audit  │  │  Checkpoint     │
  │  Store          │  │  Cache          │
  └─────────────────┘  └─────────────────┘
```

## Data Flow

1. **Request Ingress** — User submits prompt and team selection via dashboard.
2. **Auth Check** — Identity validated on every request.
3. **Budget Check** — Real-time team spend checked against monthly limit.
4. **Approval Routing** — If over budget, request is queued as `PENDING_APPROVAL` and an alert is dispatched.
5. **Agent Orchestration** — Engine decomposes task into Research → Critic → Synthesis nodes.
6. **Trace Emission** — Every node execution is logged with duration, tokens, and cost.
7. **Anomaly Detection** — Z-score engine evaluates every event. Flags cost spikes, latency spikes, token abuse, and high error rates.
8. **Response & Update** — Result returned to user; dashboard updates in real time.
9. **State Persistence** — Checkpointed agent state stored for resume and retry on failure.

## Governance Layer

| Component | Responsibility |
|---|---|
| Identity Middleware | Verification on every request |
| Budget Service | Real-time balance check per team |
| Approval Service | Queue over-budget requests for human review |
| Alert Dispatcher | Notify approvers when action is required |
| Anomaly Engine | Z-score detection across cost, latency, and token dimensions |
| Audit Logger | Immutable record of every event and decision |
| Trace Store | Optional external observability streaming |
