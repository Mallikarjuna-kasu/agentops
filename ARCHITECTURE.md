# Architecture

## System Overview

```
┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│   Dashboard UI        │────▶│  Governance Backend   │────▶│  State & Budget      │
│   React + TypeScript  │     │  FastAPI + asyncpg    │     │  Store (PostgreSQL)  │
│   Recharts + Tailwind │◄────│                       │◄────│                      │
└──────────────────────┘     └──────────────────────┘     └──────────────────────┘
                                        │
                          ┌─────────────┴─────────────┐
                          │                           │
                          ▼                           ▼
               ┌──────────────────┐       ┌──────────────────┐
               │  Budget Guard    │       │  Anomaly Engine  │
               │  check_budget()  │       │  Z-score on      │
               │  per team/call   │       │  cost/lat/tokens │
               └──────────────────┘       └──────────────────┘
                          │
                          ▼
               ┌──────────────────────┐
               │  Agent Orchestration │
               │  LangGraph           │
               │                      │
               │  Research Node       │
               │      ↓               │
               │  Critic Node         │
               │      ↓               │
               │  Synthesis Node      │
               └──────────────────────┘
                          │
               ┌──────────┴──────────┐
               │                     │
               ▼                     ▼
  ┌─────────────────────┐  ┌──────────────────────┐
  │  Trace & Audit      │  │  Checkpoint Cache    │
  │  Store (Langfuse)   │  │  Redis               │
  │  optional via env   │  │                      │
  └─────────────────────┘  └──────────────────────┘
```

## Data Flow

1. **Request Ingress** — User submits prompt + team selection via dashboard.
2. **Auth Check** — JWT middleware validates identity on every request.
3. **Budget Check** — `budget_guard.check_budget()` queries team's real-time spend vs monthly limit. (`budget_guard.py` — wired into agent runs; telemetry ingest is direct.)
4. **Approval Routing** — If over budget or `require_approval=true`, status set to `PENDING_APPROVAL`; Slack/email alert dispatched via `alert_service.py`.
5. **Agent Orchestration** — LangGraph decomposes task: Research → Critic → Synthesis nodes. Each node logs its own duration, tokens, and cost.
6. **Trace Emission** — Every node execution streamed to Langfuse (activated via `LANGFUSE_PUBLIC_KEY` + `LANGFUSE_SECRET_KEY` env vars; silent no-op if unset).
7. **Anomaly Detection** — Z-score engine runs on every telemetry event. Thresholds: cost >2σ (warn) / >3σ (critical); latency >2.5σ; tokens >3σ; error rate >10%.
8. **Response & Update** — Result returned to user; dashboard polls `/telemetry/stats`, `/telemetry/events`, `/telemetry/users` for live updates.
9. **State Persistence** — Redis stores checkpointed agent state for resume/retry on failure.

## Governance Layer

| Component | File | Responsibility |
|---|---|---|
| Identity Middleware | `auth/middleware.py` | JWT verification per request |
| Budget Service | `core/budget_guard.py` | Real-time balance check against team monthly limit |
| Approval Service | `api/agents.py` | Queue over-budget requests, expose approve/reject endpoints |
| Alert Dispatcher | `services/alert_service.py` | Slack webhook (set `SLACK_WEBHOOK_URL` in `.env`) |
| Anomaly Engine | `api/anomalies.py` | Z-score detection across cost, latency, token dimensions |
| Audit Logger | `db/schema.sql` | Immutable `llm_events` table — append-only telemetry |
| Langfuse Tracer | `core/langfuse_tracer.py` | Optional external trace streaming |

## Database Schema (key tables)

```sql
llm_events (
  id            SERIAL PRIMARY KEY,
  model         VARCHAR,
  tokens_prompt INTEGER,          -- up to 2.1B (enterprise batch safe)
  tokens_completion INTEGER,
  latency_ms    INTEGER,
  cost_usd      DECIMAL(12,6),    -- max $999,999.999999
  team_id       INTEGER,
  user_id       INTEGER,
  status        VARCHAR,          -- 'success' | 'error'
  created_at    TIMESTAMP
)

teams (
  id             SERIAL PRIMARY KEY,
  name           VARCHAR,
  budget_limit   DECIMAL(10,2),   -- monthly ceiling
  monthly_budget DECIMAL(10,2)
)

agent_runs (
  id           UUID PRIMARY KEY,
  agent_type   VARCHAR,           -- telemetry_analysis | anomaly_detection | recommendation | report_summarization
  status       VARCHAR,           -- completed | awaiting_approval | failed
  model        VARCHAR,
  tokens_used  INTEGER,
  cost_usd     DECIMAL(12,6),
  output_summary TEXT,
  created_at   TIMESTAMP
)
```

## Deployment

```yaml
# docker-compose.yml services
postgres:    PostgreSQL 15 — primary data store
redis:       Redis 7 — agent checkpoint cache
backend:     FastAPI on port 8000
dashboard:   Vite dev server on port 5173
```

```powershell
# Start
docker-compose up --build -d

# Re-seed with enterprise data
docker compose exec postgres psql -U agentops -d agentops -c "TRUNCATE llm_events, users RESTART IDENTITY CASCADE;"
docker compose exec backend python seed/seed_data.py

# Expected seed output
# [SEED] Done: 13 users, 44 events, 10 LLMs
# [SEED] Total cost seeded: $10,611.88
```

## Current Implementation Status

| Flow Node | Status | Notes |
|---|---|---|
| User submits event + team | ✅ Working | `POST /telemetry/ingest` |
| Budget check on agent run | ✅ Working | `budget_guard.py` called in `agents/run` |
| Budget check on ingest | ⚠ Not wired | `check_budget()` exists, not called in telemetry ingest |
| Approval gate | ✅ Working | Triggered via `require_approval=true` on agent runs |
| Slack/email alert | ⚠ Config needed | Code ready — set `SLACK_WEBHOOK_URL` in `.env` |
| LangGraph execution | ✅ Working | 4 agent types, step-level tracing |
| Langfuse streaming | ⚠ Config needed | Set `LANGFUSE_PUBLIC_KEY` + `LANGFUSE_SECRET_KEY` |
| Z-score anomaly detection | ✅ Working | Runs on every event, fires at 2σ / 3σ thresholds |
| Dashboard real-time update | ✅ Working | Polling-based, all views live |
| Team burn-down | ✅ Working | Teams page shows real-time budget usage % |
