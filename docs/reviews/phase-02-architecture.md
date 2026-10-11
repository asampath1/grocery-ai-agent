# Review 02: High-Level Architecture

- **Date:** 2026-10-11
- **Phase:** 2 — Architecture Design
- **Task:** [#4](https://github.com/asampath1/grocery-ai-agent/issues/4): Draft high-level architecture
- **Result:** [`architecture.md` v1](../architecture.md)
- **Decisions produced:** DEC-007 to DEC-010

## The four questions asked

1. **Components:** what are the boxes, and what does each one do?
2. **Ingestion timing:** fetch when a user asks, or on a schedule?
3. **Claude's data access:** tools call our HTTP API, or run Python functions directly?
4. **Raw data:** keep the original responses, or only the parsed fields?

---

## Round 1: My first draft (word for word)

> **Components**
> Grocery source: Provides product and price data; the exact source is still to be chosen.
> Scheduled ingestion worker: Fetches prices periodically, parses them, and stores both the parsed data and—if permitted—the original response.
> PostgreSQL: Stores products, prices, fetch timestamps, and retained raw responses so data can be queried and re-parsed.
> FastAPI backend: Serves user requests, applies business rules, queries the database, and coordinates Claude tool calls.
> Claude API: Interprets a user's question and requests backend tools when it needs product or price data.
> Next.js frontend: Lets the user ask questions and displays the answer.

### Coach feedback: strengths

- A **separate scheduled worker**, so source failures and rate limits never reach the user. The most important call in the design.
- Raw responses kept **"if permitted"**: data rights considered at the design stage.
- **Claude requests tools; the backend owns the data.** Correct division of power.
- **Fetch timestamps** stored, so freshness is visible.

### Coach feedback: challenges

| # | Challenge | Why it matters |
|---|---|---|
| 1 | "Claude API" isn't a component we build. The **agent loop** inside FastAPI is. | The loop *is* the agent, and the draft hid it. |
| 2 | Q3 wasn't answered explicitly. Are tools HTTP calls or in-process functions? | It determines testing, security, and complexity. |
| 3 | The worker and the API both need the same models. One codebase or two? | It decides the `backend/` layout. |
| 4 | "Business rules" was vague. Name one. With scheduled data, what's the **freshness rule**? | Scheduled ingestion means data is always somewhat old. |
| 5 | Where does **product matching** happen: at ingestion or at query time? | It shapes the Phase 4 schema even though matching is Phase 8. |
| 6 | The frontend is Phase 10. Until then the client is Swagger or a CLI. | The diagram should reflect what will actually exist. |
| 7 | Raw storage location (JSONB vs. files) can wait until Phase 4. | Depends on whether the source returns JSON or HTML. |

---

## Round 2: My revised answers (word for word)

> **Ingestion timing:** Scheduled, to keep user requests fast and avoid repeated calls to the source. Provisionally refresh every 6 hours and treat prices older than 12 hours as stale—show their age, but don't call them the current cheapest.
>
> **Claude's data access:** (b)—run Python tool functions in-process inside FastAPI's agent loop. Option (a) adds an unnecessary HTTP hop, extra failure points, and duplicate endpoint concerns.
>
> **Code layout:** One Python package started in two ways (api and ingest), so both share models and parsing code.
>
> **Product matching:** At ingestion time, linking source products to canonical products before querying; uncertain matches should remain flagged rather than silently linked.

### Coach feedback: all four accepted, each with a refinement

| Answer | Refinement added | Outcome |
|---|---|---|
| Scheduled, 6h / 12h stale | 12h = 2× the interval, so one failed run is tolerated. **Measure** the real price-change rate in Phase 5. Enforce the rule **in code**, not by Claude. | [DEC-007](../decision-log.md#dec-007--scheduled-ingestion-with-a-freshness-rule) |
| In-process tools | Add a **service layer** that both REST and tools wrap: one implementation. Tools are a **fixed allowlist**; no SQL tool. | [DEC-008](../decision-log.md#dec-008--claude-tools-run-in-process) |
| One package, two entry points | In Docker: **one image, two containers**. Worker trigger to be decided in Phase 5. | [DEC-009](../decision-log.md#dec-009--one-backend-package-two-entry-points) |
| Match at ingestion, flag uncertain | Make matching a **separate pipeline step** after storage, so it can be re-run over history. Keep source listings and canonical products in separate tables. | [DEC-010](../decision-log.md#dec-010--product-matching-at-ingestion-as-a-separate-step) |

---

## What improved

| Before (Round 1) | After (v1) |
|---|---|
| 6 components; Claude shown as a box we build | 9 components, separating what we build from external services |
| No explicit agent | **Agent loop** named as the agent |
| "FastAPI coordinates tool calls" | **Service layer** shared by REST endpoints and tools; tools are a fixed allowlist |
| "Business rules" | A concrete **freshness rule** (6h / 12h), enforced in code |
| Ingestion as one step | A **5-step pipeline** (fetch → raw → parse → listing → match), each step re-runnable |
| Matching not placed | Matching at ingestion, uncertain matches flagged |
| Next.js as the client | Swagger UI/CLI first, Next.js from Phase 10 |

**My main lesson:** A first draft that lists boxes isn't a design yet. Naming what *triggers* each piece, what *rules* it enforces, and what happens when it *fails* is what makes it one.
