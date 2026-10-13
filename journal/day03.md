# Day 03

## Date

2026-10-13 (Tuesday)

## What Was Completed

- Phase 3, Task 3.1 (#6): surveyed grocery data sources for 560016.
- **Blinkit:** full DevTools inspection. Found the search API (`/v1/layout/search`), product JSON (price, MRP, unit, inventory, state, group_id, merchant_id), SSR-embedded JSON, and robots.txt rules.
- **Blinkit, BigBasket, JioMart:** read the ToS. All three prohibit collecting prices with automated tools.
- **Amazon:** found the official Creators/PA API; eligibility requires an Associates account and ≥10 sales in 30 days.
- Created `docs/data-sources.md` as the running reference for all evaluated sources (DEC-014).
- **data.gov.in Mandi API:** registered, got an API key, made my first authenticated requests. Found **Bengaluru APMC tur dal, modal ₹11,800/q ≈ ₹118/kg** (vs Blinkit retail ₹165).
- Wrote Review 04 and added evidence to DEC-003 and a revisit note to DEC-004.

## What I Learned

- How to read the Network tab: filter out noise (`-collect -events -google`), and don't trust the Initiator column when a library like Sentry wraps `fetch`.
- Blinkit uses **coordinates → dark store (`merchant_id`)**, not pincode.
- Real JSON is messy: the same field has different types (`14489` vs `"14489"`), and there are 3 ways to say "empty" (`""`, `null`, `"#-NA"`).
- Parse from data the app depends on to work (the `cart_item` block), not from tracking or layout data.
- Two pack sizes = two products (`305650` = 500 g, `39493` = 1 kg), linked by `group_id`.
- robots.txt is for crawlers; the **ToS is what governs me**, and it outweighs robots.txt.
- **Check the terms first.** I spent about 40 minutes on network inspection; the ToS decided the question in 2.
- A designed API (Amazon `offersV2`) and a reverse-engineered UI response are very different: versioning, explicit currency, structured unit price, selectable fields.
- An official API with a spec (OpenAPI) and declared types makes parsing far easier than reverse-engineered JSON.
- Mandi data: unique key ≈ (date, market, commodity, variety, grade); prices mostly ₹/quintal but not always; names contain synonyms and typos.
- `curl -g` disables URL globbing so `filters[state]` works.
- Credential rotation: two key sets per app, so keys can be swapped without downtime.

## Problems Encountered

- The original project vision (compare prices across Blinkit/BigBasket/JioMart in my own database) is **prohibited by all three platforms' terms**.
- Amazon's API is blocked by eligibility (it needs affiliate sales).
- 🚨 Put the API key in `.env.example` (committed file). Caught before committing and moved to `.env`. The key was also exposed in a screenshot and in shell history, so rotate it on 2026-10-14.
- The coach's Python one-liner had a quoting bug, fixed with `%` formatting.
- A truncated paste briefly made it look as though tur dal wasn't reported.

## Decisions Made

- Policy principle: if the ToS prohibits it, we don't automate it (to be formalised in DEC-003).
- Blinkit, BigBasket, and JioMart are ruled out as automated sources.
- DEC-014: Keep a `docs/data-sources.md` reference

## Open Questions

- Adopt the data.gov.in Mandi API formally (DEC-003)? Read and quote the GODL terms first.
- Is history available (past `arrival_date`), or only today's snapshot?
- Units per commodity (coconut looks like per 1,000 nuts).
- How much should the project vision change: commodity prices, personal spending, or simulated multi-store data?

## Next Milestone

Task 3.3: Read GODL, settle DEC-003, decide the project direction and MVP question, then close Phase 3.
