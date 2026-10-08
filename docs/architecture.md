# Architecture

> Status: **Draft placeholder.** This gets designed properly in Phase 2. Only the target pipeline is recorded here for now.

## Target pipeline

```
┌──────────────┐   ┌───────────────┐   ┌──────────────┐   ┌────────────┐   ┌──────────┐   ┌────────┐
│ Live Source  │ → │ API / Scraper │ → │ Transform /  │ → │ PostgreSQL │ → │ Claude   │ → │ Answer │
│ (grocery     │   │ (ingestion)   │   │ Normalise    │   │            │   │ (tools)  │   │        │
│  platform)   │   │               │   │              │   │            │   │          │   │        │
└──────────────┘   └───────────────┘   └──────────────┘   └────────────┘   └──────────┘   └────────┘
                                                                ▲
                                                     FastAPI serves data/tools
```

## Components (to be defined in Phase 2)

- Ingestion
- Storage
- API (FastAPI)
- AI layer (Claude with tool calling)
- Frontend (Next.js)

## Open questions for Phase 2

- Should ingestion run inside the FastAPI process, or as a separate scheduled job?
- Does Claude call our API over HTTP, or call Python functions directly (in-process tools)?
- How and where are raw source responses stored for re-parsing?
