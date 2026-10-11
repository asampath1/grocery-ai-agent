# Decision Log

Each entry records a significant decision: context, options, choice, and consequences.
Format: lightweight ADR (Architecture Decision Record).

---

## DEC-001 — Load the coach prompt via CLAUDE.md

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** Claude Code automatically loads `CLAUDE.md` at session start, but not arbitrary files like `coach-prompt.md`. Without it, the coaching rules would be missing unless pasted in every session.
- **Options:**
  1. Paste the prompt manually each session. Easy to forget.
  2. Rename `coach-prompt.md` to `CLAUDE.md`. Loses the descriptive name.
  3. Keep a small `CLAUDE.md` that imports `@coach-prompt.md`. ✅
- **Decision:** Option 3.
- **Consequences:** `CLAUDE.md` is added to the repo structure. Coaching rules apply in every session automatically.

---

## DEC-002 — Revise phase order

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** The original roadmap had no data-acquisition phase even though real integrations are Learning Priority 1. It also put the Frontend MVP before the Claude integration, even though the MVP needs Claude reasoning and the frontend is Priority 5.
- **Changes:**
  - Added **Phase 3: Data Source Investigation**, placed before Database Design so the schema reflects real data.
  - Added **Phase 5: Data Ingestion MVP**.
  - Merged "Claude API Integration" and "Natural Language Querying" into one phase. NL querying through tool calling *is* the first meaningful Claude integration.
  - Moved **Frontend** to after Claude integration and price comparison.
  - The total went from 12 to 13 phases.
- **Trade-off:** There's no visual demo until Phase 10. This is acceptable because the learning priorities rank the pipeline and AI work above the UI, and progress can be demoed from the terminal or Swagger UI.

---

## DEC-003 — Data access policy (scraping vs. official APIs)

- **Date:** 2026-10-08
- **Status:** ⏳ **Open**, to be decided during Phase 3
- **Context:** Major Indian grocery platforms (Blinkit, Zepto, Swiggy Instamart, BigBasket, JioMart, Amazon Fresh) don't appear to offer public pricing APIs, and their terms of service generally restrict automated access. Prices are pincode-specific.
- **Options to evaluate:**
  1. Official or partner/affiliate APIs, where they exist
  2. Calling the internal JSON endpoints the platform's own web app uses
  3. HTML scraping
  4. Manual or semi-manual data capture
  5. Mock data, as a last resort only
- **Decision criteria:** legality/ToS, stability, learning value, effort.

---

## DEC-004 — Location scope

- **Date:** 2026-10-08
- **Status:** Accepted
- **Decision:** All price lookups target **pincode 560016 (Bangalore)**.
- **Reason:** Quick-commerce prices and stock depend on pincode. Fixing one location keeps data comparable and the scope small. The pincode is stored per price record so more locations can be added later.

---

## DEC-005 — Docker runtime on macOS: OrbStack

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** Docker on macOS needs a Linux VM. The options were Docker Desktop, OrbStack, and Colima.
- **Decision:** OrbStack.
- **Reasons:** Lightest on resources, native Apple Silicon support, very little setup, free for personal use.
- **Trade-offs:** Closed source and from a small company. If that becomes a problem, Colima (MIT licence) is a drop-in replacement, because containers and Compose files don't depend on the runtime.
- **Note:** OrbStack starts with a 30-day Pro trial. After it ends, choose the free personal-use tier. No Pro features are needed.

---

## DEC-006 — Task tracking with GitHub Issues + Projects

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** Progress was tracked only in `PROJECT_STATUS.md`. GitHub Issues and Projects provide task tracking linked to commits, which is standard team practice and visible in the portfolio.
- **Decision:** Phases are Milestones, tasks are Issues, and a Board-view Project shows the flow. Commits close issues with `closes #N`.
- **Avoiding duplication:** `PROJECT_STATUS.md` holds the phase-level summary only. Task-level detail lives in Issues.
- **Trade-off:** About 2 minutes of overhead per task, in exchange for traceable history (plan → issue → commit → close) and practice with a real-team workflow.

---

## DEC-007 — Scheduled ingestion with a freshness rule

- **Date:** 2026-10-11
- **Status:** Accepted
- **Context:** Prices can be fetched when a user asks, or fetched on a schedule and stored.
- **Options:** (a) fetch on request: always fresh, but slow, fragile, and calls the source repeatedly. (b) scheduled fetch: fast and gentle on the source, but the data is always somewhat old. ✅
- **Decision:** Scheduled ingestion every ~6h (provisional). Prices older than 12h are stale: shown with their age, never presented as the current cheapest.
- **Reasoning:** Keeps user requests fast and isolated from source failures, and lowers the risk of being blocked. The stale threshold is twice the refresh interval, so one failed run is tolerated.
- **Consequences:** The freshness rule is enforced in the service layer, not by Claude. Re-tune the interval in Phase 5 using observed price-change frequency.

---

## DEC-008 — Claude tools run in-process

- **Date:** 2026-10-11
- **Status:** Accepted
- **Options:** (a) tools make HTTP calls to our own FastAPI endpoints. (b) tools call Python service functions directly inside the agent loop. ✅
- **Reasoning:** (a) adds an unnecessary network hop, extra failure points, and duplicated concerns (auth, serialisation) for no benefit within one process.
- **Consequences:** Logic lives in a service layer that both REST endpoints and tools wrap. Tools are a fixed allowlist with validated parameters; there's no free-form SQL tool.

---

## DEC-009 — One backend package, two entry points

- **Date:** 2026-10-11
- **Status:** Accepted
- **Decision:** A single Python package under `backend/`, started as `api` (FastAPI) or `ingest` (worker).
- **Reasoning:** Both need the same models and parsing code. Two codebases would duplicate them and let them drift apart.
- **Consequences:** In Docker, one image runs as two containers with different commands. The worker trigger is decided in Phase 5.

---

## DEC-010 — Product matching at ingestion, as a separate step

- **Date:** 2026-10-11
- **Status:** Accepted
- **Options:** (a) match at query time. (b) match at ingestion time and store the links. ✅
- **Decision:** Match after a listing is stored, as its own pipeline step. Uncertain matches are flagged, never silently linked.
- **Reasoning:** Queries stay simple and fast, and wrong matches are visible rather than hidden. Because matching is separate from fetching, a matching failure never loses source data, and matching can be re-run over history when the matcher improves (Phase 8).
- **Consequences:** The Phase 4 schema keeps source listings and canonical products separate, with a link table that records match status and confidence.

---

## DEC-011 — Keep design review records

- **Date:** 2026-10-11
- **Status:** Accepted
- **Context:** The decision log records final decisions, but not how they were reached: my first answers, the coach's challenges, and the revisions. Claude Code's local conversation history isn't in the repo and is cleaned up automatically.
- **Options:** (a) do nothing. (b) commit raw transcripts: complete, but noisy and containing personal details. (c) curated review records. ✅
- **Decision:** Add `docs/reviews/`, one record per review, with first answers and revisions quoted word for word. A private `/export` backup stays optional and is not committed.
- **Consequences:** The repo structure gains `docs/reviews/`. The coach prompt now requires a review record after every review. Records 01 and 02 were backfilled.

---

## DEC-012 — Ingestion observability: run records + dead man's switch

- **Date:** 2026-10-11
- **Status:** Accepted (service choice deferred to Phase 5)
- **Context:** Exit-review question: "The source is down for 18 hours. Who notices?" In the v1 design, nobody did. Logs alone are passive.
- **Options:** (a) logs only: they explain failures but don't detect them. (b) alert on failure: misses the case where the worker never runs. (c) run records + a success ping to a dead man's switch. ✅
- **Decision:** Record each run in `ingestion_runs`, expose the last-success age on `/health`, and ping an external check after every successful run. A missing ping triggers an alert.
- **Reasoning:** Only the *absence of success* catches every failure mode, including a crashed scheduler or a stopped container. Free tiers of hosted checks (e.g. Healthchecks.io) cost nothing at this scale.
- **Also recorded from this review:** `parser_version` on parsed rows, idempotent re-runs, and four layers of tool protection (allowlist → validation → parameterised queries → least-privilege DB user). See `architecture.md`.

---

## DEC-013 — Partial failures: per-item isolation and availability status

- **Date:** 2026-10-11
- **Status:** Accepted (thresholds deferred)
- **Context:** My question: what if a run succeeds for some areas and fails for others? Pincode is stored per record (DEC-004), but partial runs hadn't been designed for.
- **Decision (now):**
  1. **Unit of work = (source, pincode, product).** Each fetch succeeds or fails independently, and a failure never aborts the run.
  2. **`availability` status** on snapshots (`in_stock` / `out_of_stock` / `not_serviceable`), kept separate from fetch errors.
  3. **Run status** is `success` / `partial` / `failed`, with item counts in `ingestion_runs`.
- **Deferred:** the alert threshold for partial runs (e.g. <80% success counts as a failure) and per-area alerting, until a second pincode or source exists.
- **Reasoning:** Treating "not available" as an error causes false alerts. Treating a failed fetch as "not available" gives users wrong answers. Isolation and the availability status cost almost nothing now and are expensive to retrofit. Thresholds would be over-engineering with one pincode.
- **Consequences:** The Phase 4 schema needs an `availability` column on snapshots and per-run item counts.
