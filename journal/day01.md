# Day 01

## Date

2026-10-08 (Thursday)

## What Was Completed

- Reviewed the coach prompt with Claude and answered its clarifying questions.
- Revised the phase plan from 12 to 13 phases (DEC-002).
- Set up `CLAUDE.md` so the coach prompt loads automatically (DEC-001).
- Scaffolded the repository folder structure and initial docs.
- Initialised git, created the public GitHub repo `asampath1/grocery-ai-agent`, and pushed the first commit (`d31e3d2`).
- Checked the local toolchain: Python 3.12.4, Node 26.8.2, and git 2.45.2 are present.
- Installed OrbStack. Docker 29.4.0 and Compose v5.1.2 are working, and `hello-world` ran natively on arm64 (DEC-005).
- Set up GitHub Milestones (Phases 1–3) and the Project board. Backfilled Phase 1 as issues #1–#3 and queued #4 (Task 2.1).
- **Phase 1 complete.**

## What I Learned

- Claude Code only auto-loads `CLAUDE.md`. Other files need to be imported with `@path`.
- Asking "what does Claude actually add?" matters. "Cheapest product" is a SQL query. Claude's value is in understanding questions and matching products.
- "Not found" from an API can mean "not allowed". Check auth scopes before assuming the resource is missing.
- GitHub Projects is a board over Issues. Writing `closes #N` in a commit links the work to the plan and closes the issue automatically.
- On a Mac, Docker always runs inside a Linux VM. The runtime only manages that VM, so containers stay portable to the VPS.
- `.env.example` is committed on purpose as a template; `.env` is ignored. The `!.env.example` rule in `.gitignore` re-includes it.
- Data access is probably the hardest part of this project, because Indian grocery platforms don't appear to have public pricing APIs.

## Problems Encountered

- `gh issue create --project` failed with "'grocery-ai-agent roadmap' not found". **Cause:** the gh token lacked the `project` scope, and GitHub reports resources you can't access as "not found". **Fix:** `gh auth refresh -s project`.

## Decisions Made

- DEC-001: Load the coach prompt via CLAUDE.md
- DEC-002: Revised phase order (data investigation before DB design; Claude before frontend)
- DEC-004: Scope is pincode 560016
- DEC-005: OrbStack as the Docker runtime
- DEC-006: Track tasks with GitHub Issues + Projects; phases map to Milestones
- Claude writes the docs; I write the code.
- One journal file per calendar day.

## Open Questions

- DEC-003: What scraping/data-access policy is acceptable?
- Is my Hostinger "Premium" plan shared hosting or a VPS?

## Next Milestone

Phase 2, Task 2.1: draft the high-level architecture.
