# Data Sources

A reference for every data source we've evaluated: its status, how it delivers data, and what its terms allow.
Kept up to date as sources are added. *How* each was investigated lives in [docs/reviews/](reviews/).

**Policy (DEC-003):** If a source's terms prohibit it, we don't automate it, whatever robots.txt or the technology allows.

## Status summary

| Source | Type | Status | Blocker | Last checked | Evidence |
|---|---|---|---|---|---|
| Blinkit | Quick-commerce | ❌ Ruled out | ToS prohibits price collection and bots | 2026-10-13 | [Review 04](reviews/phase-03-source-survey.md) |
| BigBasket | E-grocery | ❌ Ruled out | ToS prohibits automated *and* manual monitoring | 2026-10-13 | [Review 04](reviews/phase-03-source-survey.md) |
| JioMart | E-grocery | ❌ Ruled out | ToS prohibits price collection and own database | 2026-10-13 | [Review 04](reviews/phase-03-source-survey.md) |
| Amazon (Creators / PA API) | Official API | ⏸ Blocked | Needs Associates account + ≥10 sales / 30 days | 2026-10-13 | [Review 04](reviews/phase-03-source-survey.md) |
| Govt. retail prices (Dept. of Consumer Affairs) | Open data | 🔍 To evaluate | — | — | Task 3.2 |
| My own invoices / order history | Personal data | 🔍 To evaluate | — | — | Task 3.2 |

Status key: ✅ In use · 🔍 To evaluate · ⏸ Blocked (could change) · ❌ Ruled out

---

## Blinkit

- **Status:** ❌ Ruled out (ToS)
- **Terms:** "…any collection, use or derivative use of … product listings, descriptions, or prices; … or any use of data mining, robots, or similar data gathering and extraction tools."
- **robots.txt:** product pages (`/prn/…/prid/…`) allowed; search page (`/s/*`) disallowed; sitemap allowed; `/v1/` not listed.
- **Technical notes** (kept for learning reference; not to be automated):

| Aspect | Detail |
|---|---|
| Location model | Coordinates (map pin) → dark store `merchant_id`. Pincode isn't used. |
| Delivery | Client-side JSON (`GET /v1/layout/search?...`, product by ID) plus JSON embedded in SSR HTML |
| Response style | Server-driven UI (layouts, widgets) with product data embedded |
| Key fields | `product_id`, `group_id`, `merchant_id`, `name`, `brand`, `unit` ("1 kg"), `price`, `mrp`, `inventory`, `state`, `price_per_unit` |
| Data quality | Same field with different types (`14489` / `"14489"`); empty as `""` / `null` / `"#-NA"` |
| Most stable block | `cart_item` (used by the cart, so less likely to change) |
| Auth / friction | None seen for browsing |

## BigBasket

- **Status:** ❌ Ruled out (ToS)
- **Terms (10.9):** "You shall not use any 'deep-link', 'page-scrape', 'robot', 'spider' or other automatic device, program, algorithm or methodology, or any similar or equivalent manual process, to access, acquire, copy or monitor any portion of the Platform or any content…"
- **Note:** Prohibits **manual** monitoring too.
- **Technical:** not inspected (terms checked first).

## JioMart

- **Status:** ❌ Ruled out (ToS)
- **Terms (5.2):** the licence excludes "(b) any collection and use of product listings, description, or prices, … (e) any use of data mining, robots, or similar data gathering and extraction tools to extract (whether once or many times) … (e) creating and/ or publishing your own database that features parts of JioMart."
- **Note:** Prohibits storing its data in **our own database**, however it was obtained.
- **Technical:** not inspected (terms checked first).

## Amazon: Creators API (includes PA API)

- **Status:** ⏸ Blocked (eligibility). Could be revisited if the eligibility rules change or the app gets real users.
- **Portal:** `affiliate-program.amazon.in/creatorsapi`
- **Eligibility:** approved Associates (creators) account **and** ≥10 qualifying sales in the past 30 days. Access review takes up to 48h; until then calls return `AssociateNotEligible`.
- **Auth:** up to 2 applications, each with 2 credential sets (intended for rotation).
- **Request:** `partnerTag`, `keywords` / `searchIndex` / `brand`, `itemCount`, `resources` (choose the fields returned).
- **Response fields:** `asin`, `itemInfo.title`, `offersV2.listings[].price.money.{amount,currency}`, `price.pricePerUnit`, `price.savingBasis` (list price), `price.savings`, `availability.type`, `merchantInfo.name`.
- **Open questions:** Are prices location-specific (Amazon Fresh)? Do the licence terms limit caching, or showing prices next to other retailers?
