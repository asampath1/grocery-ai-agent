# Review 03: Phase 2 Exit Review (Understanding Check)

- **Date:** 2026-10-11
- **Phase:** 2 — Architecture Design
- **Task:** [#5](https://github.com/asampath1/grocery-ai-agent/issues/5): Phase 2 exit review
- **Format:** four scenario questions answered **without looking at the docs**
- **Verdict:** ✅ Ready to proceed to Phase 3
- **Decisions produced:** DEC-012; v1.1 refinements to `architecture.md`

---

## Q1: The source is down for 18 hours. What happens and what does the user see?

**My first answer (word for word):**
> Source down for 18 hours: Fails and latest prices become stale under the 12-hour rule showing any last-known prices with the time.

**Coach challenge:**
- Correct, but say explicitly that **the user's request still succeeds**. Only the worker fails; the API reads from Postgres. That's the point of DEC-007.
- Staleness reaches the user through `is_stale` / `fetched_at` in the tool result. The code decides, and Claude puts it into words.
- **Design gap found:** who *notices* the source is down? In v1, nobody did.

**Follow-up:** "What's the simplest way you could find out the worker is failing?"

**My answer (word for word):**
> I asssumed there will be a log since we cannot manually check the prices for all the pincodes.

**Coach challenge:** Logs explain *why* something failed, but they don't detect *that* it failed. Nobody reads them until they're already investigating. Detection needs (1) run records queryable via `/health` and (2) a **dead man's switch** that alerts when a *success* ping is missing. That's the only way to catch a worker that never runs at all.

**Outcome:** [DEC-012](../decision-log.md#dec-012--ingestion-observability-run-records--dead-mans-switch). Pipeline step 6 ("record run") added. Failure-handling table added to `architecture.md`.

---

## Q2: Claude calls `find_cheapest` with `pincode="DROP TABLE"`. What stops it?

**My answer (word for word):**
> Invalid pincode: The tool's validated parameters should reject DROP TABLE as an invalid pincode before it reaches the service layer. The fixed tool allowlist and lack of free-form SQL prevent Claude from issuing database commands.

**Coach feedback:** ✅ Good. Two more independent layers added:
1. Allowlist *(mine)*
2. Validation `^\d{6}$` plus supported pincode *(mine)*
3. **Parameterised queries:** the real SQL-injection defence, even if validation is bypassed
4. **Least-privilege DB user:** the API can't `DROP` anything

**Key lesson:** Claude's tool arguments are **untrusted input**, because they may reflect what a user typed to steer Claude.

**Outcome:** "Security: layered protections" section added to `architecture.md`.

---

## Q3: A pack-size parser bug has been running for three weeks. What can you do?

**My answer (word for word):**
> Pack-size parser bug: Fix the parser, then re-run parsing against retained raw responses to correct historical listings.

**Coach feedback:** ✅ Correct. This is enabled by keeping raw data plus the step-by-step pipeline (DEC-010). Refinements:
- Store **`parser_version`** on parsed rows, so the bad rows can be found and re-parsed selectively.
- Re-runs must be **idempotent** (upserts), so they correct rows rather than duplicate them.
- **Caveat:** only possible where keeping raw data is permitted. This feeds into DEC-003.

**Outcome:** Added to the ingestion section of `architecture.md`. To be applied in the Phase 4 schema.

---

## Q4: "Where's the agent in your system?"

**My answer (word for word):**
> The agent is the loop inside FastAPI that sends the question and tool definitions to Claude, runs Claude's requested tools, returns their results, and repeats until Claude responds.

**Coach feedback:** ✅ Accurate. Shorter version for interviews:
> "Claude decides which tool to call; my agent loop in FastAPI runs it. The loop is the agent, Claude is the reasoning inside it."

---

## My follow-up question: "will there be a scenario, it runs for few areas and not running for other areas. how it works."

**Context:** This came up after the exit review. The design stores the pincode per record, but hadn't considered a run that **partly** succeeds.

**Coach response:**
- This is a **partial failure**, among the most common situations in real pipelines (rate limits mid-run, timeouts, areas the platform doesn't serve).
- **Unit of work = (source, pincode, product)**, so each item succeeds or fails independently.
- Freshness is **already per area** thanks to the per-snapshot `fetched_at` (DEC-007).
- **"Fetch failed" ≠ "not available".** The first means we don't know the price; the second is a fact about the area and should be stored as data. Mixing them up causes false alerts or wrong answers.
- Runs have three outcomes: `success` / `partial` / `failed`. Alerting needs a threshold.
- **Build now only what's cheap now and hard to add later:** per-item isolation and an `availability` status. Defer thresholds and per-area alerting.

**My answer:** "yes, add it as DEC-013"

**Outcome:** [DEC-013](../decision-log.md#dec-013--partial-failures-per-item-isolation-and-availability-status)

---

## What improved

| Area | Before the exit review | After |
|---|---|---|
| Failure detection | Not designed ("there will be a log") | Run records + `/health` + dead man's switch |
| Tool security | 2 layers (allowlist, validation) | 4 layers, and the principle that model output is untrusted input |
| Re-processing | "Re-run the parser" | Versioned, idempotent, selective re-parse |
| Failure modes | Implicit | Explicit failure-handling table |
| Partial runs | Not considered | Per-item isolation; `availability` kept separate from fetch errors; 3-state run status |

**My main lesson:** Walking failure scenarios through the design found a gap (failure detection) that drawing the components never showed.
