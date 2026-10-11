# Architecture

> Status: **v1.1, agreed 2026-10-11** (Phase 2). Decisions: DEC-007 to DEC-013.
> Revisit at the end of Phase 3 (data source chosen) and Phase 4 (schema designed).

## System overview

```
                         ┌──────────────────────────── one Python package (backend/) ───────────────────────────┐
                         │                                                                                       │
┌──────────────┐  fetch  │  ┌─────────────────────────┐            ┌────────────────────────────────────────┐   │
│ Grocery      │◄────────┼──│ Ingestion worker        │            │ FastAPI app                            │   │
│ source       │ ───────►│  │ (scheduled, every ~6h)  │            │                                        │   │
│ (TBD, Ph. 3) │ response│  │ fetch → raw → parse →   │            │  REST endpoints ──┐                    │   │
└──────────────┘         │  │ listing → match         │            │                   ▼                    │   │
                         │  └───────────┬─────────────┘            │  Agent loop ──► Tool functions         │   │
                         │              │ writes                   │      ▲  │            │                │   │
                         │              ▼                          │      │  │            ▼                │   │
                         │  ┌───────────────────────────────────────┴──────┼──┼─── Service layer ─────────┐ │   │
                         │  │                         PostgreSQL           │  │   (find_cheapest, …)      │ │   │
                         │  │  raw responses · source listings · price     │  │   reads                   │ │   │
                         │  │  snapshots · canonical products · matches    │  │                           │ │   │
                         │  └──────────────────────────────────────────────┼──┼───────────────────────────┘ │   │
                         └─────────────────────────────────────────────────┼──┼──────────────────────────────┘   
                                                                           │  │ messages + tool definitions
                                                                           │  ▼
                                                                    ┌──────────────┐
                    ┌──────────────────────────────┐   question     │  Claude API  │
                    │ Client                       │──────────────► │  (external)  │
                    │ Ph. 6–9: Swagger UI / CLI    │◄── answer ──── └──────────────┘
                    │ Ph. 10+: Next.js frontend    │   (via FastAPI)
                    └──────────────────────────────┘
```

Simplified view:

```
Source ──► Ingestion worker ──► PostgreSQL ◄── Service layer ◄── Tools ◄── Agent loop ◄──► Claude
                                                     ▲                         ▲
                                              REST endpoints              FastAPI ◄── Client
```

## Components

| Component | Built by us? | Responsibility | Triggered by |
|---|---|---|---|
| **Grocery source** | No | Provides product and price data for pincode 560016. Chosen in Phase 3. | Our worker |
| **Ingestion worker** | Yes | Runs the pipeline below on a schedule. Keeps source failures and rate limits away from users. | Timer (~6h) |
| **PostgreSQL** | No (we design the schema) | Stores raw responses, source listings, price snapshots with timestamps, canonical products, and match links. | — |
| **Service layer** | Yes | The business logic: lookups, comparisons, freshness rule. The **single implementation** shared by REST and tools. | API / tools |
| **REST endpoints** | Yes | Thin HTTP wrappers over the service layer. Used for testing, Swagger, and later the frontend. | Client |
| **Tool functions** | Yes | Thin wrappers over the service layer, exposed to Claude as a **fixed allowlist** with validated parameters. | Agent loop |
| **Agent loop** | Yes | Sends the question and tool definitions to Claude, runs the requested tools, returns results, repeats until Claude answers. *This is "the agent."* | Client request |
| **Claude API** | No | Interprets the question, chooses tools, writes the answer from tool results. Has **no direct database access**. | Agent loop |
| **Client** | Yes (later) | Swagger UI / CLI in Phases 6–9; Next.js from Phase 10. | User |

## Flow 1: Ingestion (background, scheduled)

```
Timer
  ↓
1. Fetch      request source for product X at pincode 560016   (rate-limited, retries with backoff)
  ↓
2. Store raw  save original response + fetched_at              (if the terms allow; location TBD in Phase 4)
  ↓
3. Parse      extract name, pack size, price, MRP, stock       (parse errors logged; raw kept)
  ↓
4. Listing    upsert source listing + insert price snapshot
  ↓
5. Match      link listing → canonical product                 (uncertain ⇒ flagged, NOT linked)
  ↓
6. Record run write ingestion_runs row (status, counts, error); on success ping dead man's switch
```

Each step can fail without losing the output of the steps before it. Steps 3–5 can be **re-run from stored raw data** when the parser or matcher improves.

- **Unit of work = (source, pincode, product).** Each item succeeds or fails on its own, and one failure never aborts the run (DEC-013).
- Snapshots carry an **`availability`** status (`in_stock` / `out_of_stock` / `not_serviceable`), kept separate from fetch errors. "Not available" is data, not a failure.
- Parsed rows record a **`parser_version`**, so a buggy version's output can be found and re-parsed selectively.
- Re-runs are **idempotent** (upserts, not inserts): running twice gives the same result as running once.
- Re-parsing history is only possible where keeping raw data is permitted (see DEC-003).

## Flow 2: Question answering (on request)

```
User: "Where is Tata Sampann Toor Dal 1kg cheapest?"
  ↓
FastAPI  →  Agent loop  →  Claude  (question + tool definitions)
                              ↓
                  tool_use: find_cheapest(product="Tata Sampann Toor Dal 1kg", pincode="560016")
                              ↓
Agent loop runs tool  →  Service layer  →  PostgreSQL
                              ↓
          result: [{store, price, fetched_at, is_stale}, …]
                              ↓
Claude  →  "Cheapest at <store> for ₹X (checked 2h ago)…"
  ↓
Client
```

## Key rules

- **Freshness (DEC-007):** refresh every ~6h (provisional). A price older than **12h is stale**. Stale prices are shown with their age but never presented as the current cheapest. The rule is enforced **in the service layer**, not by Claude. The interval is to be tuned in Phase 5 based on how often prices actually change.
- **Tools (DEC-008):** in-process Python functions, a fixed allowlist, parameters validated. No free-form SQL or arbitrary HTTP.
- **Code layout (DEC-009):** one package, two entry points (`api`, `ingest`). In Docker: one image, two containers.
- **Matching (DEC-010):** a separate step after ingestion. Uncertain matches are flagged, never silently linked. It can be re-run over historical listings.

## Failure handling

| Failure | What happens | What the user sees |
|---|---|---|
| Source down / blocked | Worker run fails and is recorded in `ingestion_runs`. No success ping, so an alert fires. **API unaffected.** | Last-known prices with their age. After 12h, marked stale. |
| Some items/areas fail in a run (partial failure) | Failed items are recorded and the rest continue. Run status is `partial`. | Fresh prices where fetched; last-known price with its age elsewhere. |
| Product not stocked / area not serviceable | Stored as an `availability` status. **Not** an error and no alert. | "Not available at <store> in your area". The old price is never shown as if it can be bought. |
| Parser bug discovered | Fix the parser, bump `parser_version`, re-parse from raw (idempotent). | Corrected prices after the re-run. |
| Worker never runs (scheduler broken, container down) | No log, no failure record, but **no success ping**, so the dead man's switch alerts. | Same as "source down". |
| Malicious or invalid tool argument | Rejected by parameter validation. Parameterised queries and a least-privilege DB user act as further layers. | A polite "can't do that" from Claude. |

## Security: layered protections for tools

Claude's tool arguments are **untrusted input**, because they may reflect what a user wrote.

1. **Fixed tool allowlist.** There's no free-form SQL or HTTP tool.
2. **Parameter validation.** E.g. pincode must match `^\d{6}$` and be a supported pincode.
3. **Parameterised queries** in the service layer. Values are always passed as data, never as SQL.
4. **Least-privilege DB user** for the API. It can read and write data but can't drop or alter tables.

## Observability (DEC-012)

- **`ingestion_runs` table:** one row per worker run, with status `success` / `partial` / `failed` and item counts.
- **`/health` endpoint:** reports the age of the last successful run.
- **Dead man's switch:** the worker pings an external check after each successful run, and the check alerts if no ping arrives in time. Tool chosen in Phase 5 (e.g. Healthchecks.io free tier).
- **Logs:** for diagnosing *why* something failed, not for detecting *that* it failed.

## Deferred decisions

| Question | Decide in |
|---|---|
| Which grocery source, and how we access it (DEC-003) | Phase 3 |
| Where raw responses are stored (Postgres `JSONB` vs. files) | Phase 4 |
| Table design for listings / products / snapshots / matches | Phase 4 |
| Worker trigger: cron vs. systemd timer vs. in-container loop | Phase 5 |
| Matching method: rules vs. Claude vs. both | Phase 8 |
| Dead man's switch service and alert channel (email / Telegram) | Phase 5 |
| Partial-failure threshold for alerting (e.g. <80% success ⇒ fail) and per-area alerting | When a 2nd pincode or source is added |
