# AEGIS-FX

Planning documents for **AEGIS-FX**, a human-in-the-loop Forex decision-support system (MT5 + Python, EURUSD first, Telegram alerts, manual execution, no auto-execution).

| Document | What it is |
|---|---|
| [`PLAN.md`](PLAN.md) | **V4 plan.** Amends the V3 master build spec: design decisions, architecture, the strategy to test first, data plan, risk and execution, the LLM's role, validation protocol, staged roadmap with go/no-go gates, and what happens to each of the 30 proposed additions. Includes 11 Mermaid diagrams: 8 flowcharts, 2 state diagrams, 1 sequence diagram. |
| [`REVIEW.md`](REVIEW.md) | **Evidence.** A five-lens adversarial review of V3 + the 30 additions: findings with numbers, fact checks, and limitations. |

No code is in this folder yet. The next step is to answer `PLAN.md` §15, then Stage 0 (pre-registration Part A), then Stages 1–2 (data foundation and research backtester). See `PLAN.md` §16. The most likely Stage 3 result is NO-GO or NOT_EVALUABLE, and the plan treats that as a valid outcome.

Nothing here claims the system is profitable, accurate, low-risk or production-ready. Those claims require the evidence described in `PLAN.md` §10.
