# Day 02

## Date

2026-10-11 (Sunday)

## What Was Completed

- Phase 2, Task 2.1 (#4): drafted the high-level architecture and answered the four design questions.
- Reviewed with the coach. All four answers were accepted, with refinements.
- `docs/architecture.md` v1 written: components, ingestion flow, question-answering flow, key rules, deferred decisions.
- Recorded DEC-007 to DEC-010.
- Started `docs/reviews/` to keep the review conversations (first answer → challenge → revision). Backfilled Review 01 (coach prompt) and Review 02 (architecture) (DEC-011).

## What I Learned

- "Claude API" isn't a component I build. The **agent loop** (send → run tool → return result → repeat) is, and it's the "agent" in the system.
- A **service layer** lets REST endpoints and Claude tools share one implementation.
- Rules that must always hold (like freshness) belong in code, not in the prompt.
- Stale threshold = 2× refresh interval, so one failed run is tolerated.
- Splitting the pipeline into steps (fetch → raw → parse → listing → match) means a failure in one step doesn't lose the others' work, and later steps can be re-run.
- Tools must be a fixed allowlist. Never give the model a free-form SQL tool.

## Problems Encountered

- None.

## Decisions Made

- DEC-007: Scheduled ingestion (~6h), 12h stale threshold, enforced in code
- DEC-008: Tools run in-process via a service layer
- DEC-009: One backend package, two entry points (`api`, `ingest`)
- DEC-011: Keep design review records in `docs/reviews/`
- DEC-010: Matching at ingestion as a separate step; uncertain matches flagged

## Open Questions

- Is 6h the right refresh interval? (Measure in Phase 5.)
- Raw response storage: Postgres JSONB or files? (Phase 4)
- DEC-003 data access policy (Phase 3)

## Next Milestone

Phase 2 exit review, then Phase 3: Data Source Investigation.
