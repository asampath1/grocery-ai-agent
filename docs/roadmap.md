# Roadmap

Time budget: about 1 hour/day, about 8 hours/week. Estimates are rough and get revised at each phase exit.

| # | Phase | Goal / Exit criteria | Est. |
|---|-------|----------------------|------|
| 1 | Repository Setup | Repo on GitHub, structure in place, docs scaffolded, toolchain checked | 1 wk |
| 2 | Architecture Design | System diagram, component responsibilities, data-flow doc | 1 wk |
| 3 | Data Source Investigation | Survey platforms for 560016: API? auth? ToS? response shape? Choose ONE source and record a sample request/response | 1–2 wk |
| 4 | Database Design | Schema for stores, products, listings, price snapshots, justified table by table; Postgres running in Docker | 1 wk |
| 5 | Data Ingestion MVP | Script fetches 1 product from 1 live source, parses it, and saves it to Postgres with errors handled and raw responses kept | 2 wk |
| 6 | Backend API MVP | FastAPI endpoints serving stored prices | 1–2 wk |
| 7 | Claude Integration & NL Querying | Claude answers "Where is X cheapest?" via tool calling over our API/DB | 2 wk |
| 8 | Product Normalisation Engine | Same product matched across sources (rules + Claude) | 2 wk |
| 9 | Multi-Source Price Comparison | 2–3 sources ingested and compared | 2 wk |
| 10 | Frontend MVP (Next.js) | Minimal chat/search UI. JS/React learned from fundamentals | 2–3 wk |
| 11 | Basket Optimiser | Cheapest basket across stores, within a budget | 2 wk |
| 12 | Deployment | Dockerised stack on a Hostinger VPS, public URL, HTTPS | 1–2 wk |
| 13 | Portfolio Documentation | README, architecture write-up, demo, lessons learned | 1 wk |

**Total:** about 20–25 weeks at 8 hrs/week.

## Why this order (summary; full reasoning in decision-log DEC-002)

- **Investigate data before designing the database.** The schema should reflect what real sources actually return.
- **Ingest data before building the API.** The pipeline is the top learning priority, and the API needs data to serve.
- **Claude before the frontend.** The MVP needs Claude reasoning, and it can be exercised from the terminal or the API without a UI.
- **Frontend late.** It's the lowest learning priority and needs JS/React taught from scratch. By then the backend is stable.
