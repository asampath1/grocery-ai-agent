# Day 03

## Date

2026-10-13 (Tuesday)

## What Was Completed

- Phase 3, Task 3.1 (#6): surveyed grocery data sources for 560016.
- **Blinkit:** full DevTools inspection. Found the search API (`/v1/layout/search`), product JSON (price, MRP, unit, inventory, state, group_id, merchant_id), SSR-embedded JSON, and robots.txt rules.
- **Blinkit, BigBasket, JioMart:** read the ToS. All three prohibit collecting prices with automated tools.
- **Amazon:** found the official Creators/PA API; eligibility requires an Associates account and ≥10 sales in 30 days.
- Created `docs/data-sources.md` as the running reference for all evaluated sources (DEC-014).
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
- Credential rotation: two key sets per app, so keys can be swapped without downtime.

## Problems Encountered

- The original project vision (compare prices across Blinkit/BigBasket/JioMart in my own database) is **prohibited by all three platforms' terms**.
- Amazon's API is blocked by eligibility (it needs affiliate sales).

## Decisions Made

- Policy principle: if the ToS prohibits it, we don't automate it (to be formalised in DEC-003).
- Blinkit, BigBasket, and JioMart are ruled out as automated sources.
- DEC-014: Keep a `docs/data-sources.md` reference

## Open Questions

- Which legitimate data source should we use? (Task 3.2)
- How much should the project vision change: commodity prices, personal spending, or simulated multi-store data?

## Next Milestone

Task 3.2: Compare the legitimate alternatives, settle DEC-003, and choose one source.
