# grocery-ai-agent

**AI Grocery Price Intelligence Platform.** An AI application that reasons over live grocery pricing data from Indian grocery platforms, built as a hands-on AI engineering learning project.

> Status: 🚧 Phase 1 — Repository Setup. See [PROJECT_STATUS.md](PROJECT_STATUS.md).

## The first question it will answer

> "Where is Tata Sampann Toor Dal 1kg cheapest?" (pincode 560016, Bangalore)

## Planned pipeline

```
Live Source → API / Scraper → FastAPI Backend → PostgreSQL → Claude (tool calling) → Answer
```

## Tech stack

| Layer      | Choice             |
|------------|--------------------|
| Frontend   | Next.js            |
| Backend    | FastAPI (Python)   |
| Database   | PostgreSQL         |
| AI         | Claude API         |
| Containers | Docker             |
| Hosting    | Hostinger VPS      |

## Repository guide

| Path | Purpose |
|------|---------|
| `docs/` | Vision, roadmap, architecture, decisions, glossary |
| `journal/` | Daily learning journal |
| `backend/` | FastAPI service and ingestion code |
| `frontend/` | Next.js app |
| `database/` | Schema and sample data |
| `prompts/` | Versioned Claude prompts |
| `docker/` | Container configuration |

## Learning journey

This repo documents the whole process: why each decision was made ([decision log](docs/decision-log.md)) and what was learned each day ([journal](journal/)).
