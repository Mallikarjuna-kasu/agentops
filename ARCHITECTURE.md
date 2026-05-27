# Architecture

## System Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   User Interface  │────▶│ Governance      │────▶│ State & Budget  │
│   (Dashboard)     │     │ Backend         │     │ Store           │
│                   │◄────│                 │◄────│                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │ Agent           │
                        │ Orchestration   │
                        │ Engine          │
                        │                 │
                        │  • Research     │
                        │  • Critic       │
                        │  • Synthesis    │
                        └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │ Trace & Audit   │
                        │ Store           │
                        └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │ Checkpoint      │
                        │ Cache           │
                        └─────────────────┘
```

## Data Flow

1. **Request Ingress** — User submits prompt via dashboard.
2. **Auth & Budget Check** — Backend validates identity, queries team budget.
3. **Approval Routing** — If over budget, status set to `PENDING`; alert dispatched.
4. **Agent Orchestration** — Engine decomposes task into research/critic/synthesis nodes.
5. **Trace Emission** — Every node execution is streamed to trace store.
6. **Response & Update** — Result returned to user; dashboard charts update.
7. **State Persistence** — Cache stores checkpointed state for resume/retry.

## Governance Layer

| Component | Responsibility |
|---|---|
| Identity Middleware | Verification per request |
| Budget Service | Real-time balance check + deduction |
| Approval Service | Queue over-budget requests for human review |
| Audit Logger | Immutable trace of every decision point |

## Deployment

- Containerized orchestration
- Async backend with persistent database
- Real-time frontend dashboard
- External observability integration
