# Day 01

## Date

2026-10-08 (Thursday)

## What Was Completed

- Reviewed the coach prompt with Claude and answered its clarifying questions.
- Revised the phase plan from 12 to 13 phases (DEC-002).
- Set up `CLAUDE.md` so the coach prompt loads automatically (DEC-001).
- Scaffolded the repository folder structure and initial docs.
- Checked the local toolchain: Python 3.12.4, Node 26.8.2, and git 2.45.2 are present. Docker and psql are missing.

## What I Learned

- Claude Code only auto-loads `CLAUDE.md`. Other files need to be imported with `@path`.
- Asking "what does Claude actually add?" matters. "Cheapest product" is a SQL query. Claude's value is in understanding questions and matching products.
- Data access is probably the hardest part of this project, because Indian grocery platforms don't appear to have public pricing APIs.

## Problems Encountered

- None yet.

## Decisions Made

- DEC-001: Load the coach prompt via CLAUDE.md
- DEC-002: Revised phase order (data investigation before DB design; Claude before frontend)
- DEC-004: Scope is pincode 560016
- Claude writes the docs; I write the code.
- One journal file per calendar day.

## Open Questions

- DEC-003: What scraping/data-access policy is acceptable?
- Is my Hostinger "Premium" plan shared hosting or a VPS?
- Docker Desktop vs. OrbStack vs. Colima on the Mac?

## Next Milestone

Finish Phase 1: initialise git, create the GitHub repo, and make the first commit and push.
