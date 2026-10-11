# Glossary

Terms are added as they come up in the project.

| Term | Meaning in this project |
|------|-------------------------|
| **ADR** | Architecture Decision Record. A short note capturing a decision, its options, and its trade-offs. See `decision-log.md`. |
| **Quick-commerce** | 10–30 minute delivery platforms (Blinkit, Zepto, Instamart). Prices are pincode-specific. |
| **Pincode** | Indian postal code. Determines which dark store serves you, and therefore the price and stock. Ours is 560016. |
| **Ingestion** | Fetching data from an external source and bringing it into our system. |
| **Normalisation** | Converting different representations of the same thing into one canonical form, e.g. "1 kg", "1000g", and "1kg" all mean the same quantity. |
| **Product matching** | Deciding that listings on different stores refer to the same real-world product. |
| **Tool calling** | A Claude API feature where the model asks the application to run a defined function (a "tool") and uses the result in its answer. |
| **Price snapshot** | A price observed for a product, at a store, at a specific time. |
| **Agent loop** | Our code that sends a question and tool definitions to Claude, runs the tools Claude requests, returns the results, and repeats until Claude gives a final answer. |
| **Service layer** | Python functions holding the business logic (e.g. `find_cheapest`). Shared by REST endpoints and Claude tools. |
| **Tool allowlist** | The fixed set of tools Claude may call, each with validated parameters. Nothing outside the list can run. |
| **Stale price** | A price snapshot older than the freshness threshold (12h). Shown with its age, never presented as the current cheapest. |
| **Source listing** | A product as it appears on one store, with that store's name and pack size. Linked to a canonical product by matching. |
| **Canonical product** | Our single, store-independent record of a real product (e.g. "Tata Sampann Toor Dal, 1 kg"). |
