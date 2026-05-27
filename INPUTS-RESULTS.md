# Inputs & Results

Real inputs and system outputs from the running POC.
All token counts and costs use 2025/2026 published enterprise API pricing.

---

## Input 1 — Standard Agent Run (Under Budget)

**User Input:**
- Team: Data Science
- Prompt: "Analyse Q3 procurement spend anomalies across AP transactions"
- Model: gemini-2.5-pro
- Estimated tokens: ~8M

**System Processing:**
- Budget check: PASS (\$61.60 estimated < \$500 remaining)
- Approval required: NO
- Agent workflow: Research → Critic → Synthesis
- Execution time: 340ms

**Results:**
- Output: Structured anomaly report with flagged vendor patterns
- Tokens used: 8,000,000 prompt + 3,200,000 completion
- Cost: **\$61.60**
- Trace: 4 spans logged (fetch → parse → llm_analysis → format)
- Dashboard: Data Science team burn updated in real time

---

## Input 2 — High-Cost Batch Job (Over Budget) — Anomaly Detected

**User Input:**
- Team: Engineering
- Prompt: "Process full document corpus — 120M tokens for compliance pattern detection"
- Model: gpt-5.5
- Estimated tokens: ~165M

**System Processing:**
- Budget check: FAIL (\$4,500 estimated >> \$2,000 monthly budget)
- Approval required: YES
- Anomaly engine: **5.8σ cost spike — CRITICAL**
- Routed to: Team Lead (Approver role)

**Results:**
- Status: PENDING_APPROVAL
- Notification: Slack + Email alert dispatched
- Dashboard: Engineering card shows 303% budget usage, red over-budget flag
- Anomaly feed: Cost Spike · critical · gpt-5.5 · \$4,500 · 5.8σ
- AI Recommendation generated:
  1. Switch to gpt-4o-mini for sub-tasks that don't require gpt-5.5 capability
  2. Add `max_tokens=4096` cap on individual API calls
  3. Batch into smaller chunks — 10M tokens/call instead of 120M
  4. Enable response caching for repeated compliance patterns

---

## Input 3 — Latency Spike Detection

**User Input:**
- Team: Engineering
- Prompt: "Generate executive summary from financial model"
- Model: claude-opus-5
- Normal expected latency: ~1,200ms

**System Processing:**
- Budget check: PASS
- Execution: completed
- Response time: **6,800ms** (4.9σ above baseline of 1,218ms avg)

**Results:**
- Output: Executive summary returned to user
- Cost: \$180.00 (4M prompt + 1.6M completion tokens)
- Anomaly engine: **Latency Spike · critical · 4.9σ**
- Alert feed entry: claude-opus-5 · 6,800ms · \$180.00
- AI Recommendation:
  1. Route executive summary tasks to claude-sonnet-5 (1,200ms avg)
  2. Enable streaming — perceived response time drops immediately
  3. Check for regional endpoint saturation at time of call
  4. Reduce prompt payload — avoid full conversation history in each call

---

## Input 4 — Token Abuse Detection

**User Input:**
- Team: Operations
- Prompt: (abnormally long — consistent with prompt injection pattern)
- Model: llama-3.3-70b
- Normal token range: 8M–16M tokens/call

**System Processing:**
- Tokens detected: **210,000,000** (prompt + completion)
- Z-score: **6.1σ** — far above 3σ critical threshold
- Classification: TOKEN_ABUSE · critical

**Results:**
- Cost: \$189.00
- Anomaly engine: Token Abuse alert fired immediately
- AI Recommendation:
  1. Enforce per-user token budget via middleware (hard cap at 20M/call)
  2. Add prompt injection guards — detect unusually long inputs before sending
  3. Set `max_tokens` limit in API call configuration
  4. Audit input length patterns for this user — check for jailbreak attempts

---

## Input 5 — Multi-Agent Workflow with Approval

**User Input:**
- Team: Marketing
- Workflow: Recommendation agent triggered after anomaly detection
- Require approval: YES
- Model: claude-opus-5

**System Processing:**
- Agent: Recommendation (LangGraph)
- Steps: fetch_anomalies → context_build → llm_recommend → human_review
- Status after llm_recommend: **AWAITING_APPROVAL**

**Results:**
- Tokens used: 6,200 (analysis phase only, action phase paused)
- Cost so far: \$0.0954
- Approval queue: item appears with risk_level=medium
- Estimated impact: "42% cost reduction if applied across Engineering team"
- Approver action: Click Approve → agent executes final step
- Full audit trail: approved by kasu.mallikarjuna · timestamp logged

---

## Summary Table

| Input | Model | Tokens | Cost | Anomaly | Action |
|---|---|---|---|---|---|
| Q3 procurement analysis | gemini-2.5-pro | 11.2M | \$61.60 | None | Executed directly |
| Compliance batch job | gpt-5.5 | 165M | **\$4,500** | Cost Spike 5.8σ | Pending approval |
| Executive summary | claude-opus-5 | 5.6M | \$180.00 | Latency Spike 4.9σ | Alert + recommendation |
| Long-form ops prompt | llama-3.3-70b | 210M | \$189.00 | Token Abuse 6.1σ | Alert + recommendation |
| Recommendation agent | claude-opus-5 | 6,200 | \$0.0954 | None | Approval gate → approved |
