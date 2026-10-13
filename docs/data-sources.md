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
| Govt. retail prices (Dept. of Consumer Affairs) | Open data | 🔍 Evaluating | Price dataset/channel not yet confirmed | 2026-10-13 | Task 3.2 |
| data.gov.in: Mandi daily prices (Agmarknet) | Open data API | 🔍 **Lead candidate**: verified working | Wholesale, not retail; GODL terms to be quoted | 2026-10-13 | [Review 04](reviews/phase-03-source-survey.md) Part 4 |
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

## Government retail prices: Dept. of Consumer Affairs, Price Monitoring Division

- **Status:** 🔍 Evaluating (Task 3.2)
- **Source vs. channels:** one source (the Price Monitoring Division collects the prices), possibly several distribution channels:

| Channel | Findings so far | Open questions |
|---|---|---|
| `fcainfoweb.nic.in` (reports) | **Bengaluru is a reporting centre** (Chennai too) | Tur/arhar dal price for Bengaluru? Format (web table / PDF / Excel)? How is the date selected? Site terms? |
| data.gov.in (datasets + API) | Has datasets back to 2019. The dataset found is **pulse allocation to states (no prices)**, the wrong one. Datasets are downloadable; the **API needs free registration (API key)** | Is there a *price* dataset (centre, commodity, date, price)? API rate limits? Licence (GODL) terms? |

- **Note:** Registration for an API key is standard API authentication, not an eligibility gate like Amazon's. Store the key in `.env` only.

## data.gov.in: Current Daily Price of Various Commodities from Various Markets (Mandi)

- **Status:** 🔍 Evaluating. **Lead candidate**: a real, official, documented API with an open licence.
- **Found:** 2026-10-13. Published 23/05/2013, **updated 13/10/2026** (updated daily). About 59k downloads.
- **Licence:** Government Open Data License – India (GODL). Terms to be read and quoted.
- **API:** `GET /resource/9ef84268-d588-465a-a308-a864a43d0070`, documented as **OpenAPI (OAS) 2.0**
- **Auth:** API key, generated after free registration. Stored in `.env` as `DATA_GOV_API_KEY`.
- **Bulk download:** form asks for usage type, purpose, and CAPTCHA (required). Name, mobile, and email appear optional. The API is preferred.
- **Request (verified):**
  ```
  GET https://api.data.gov.in/resource/9ef84268-d588-465a-a308-a864a43d0070
      ?api-key=${DATA_GOV_API_KEY}&format=json&limit=1000&filters[state]=Karnataka
  ```
  `filters[state]` works (`field_exposed` lists `state.keyword`, `district`, `market`, `commodity`, `variety`, `grade`). Use `curl -g` so `[ ]` isn't treated as URL globbing.
- **Response envelope:** `status`, `version` (2.2.0), `total`, `count`, `limit`, `offset`, `field` (declared types), `records[]`
- **Record fields:**

| Field | Type | Example |
|---|---|---|
| `state` | keyword | `Karnataka` |
| `district` | keyword | `Bengaluru` |
| `market` | keyword | `Bengaluru APMC` |
| `commodity` | keyword | `Red gram split/Arhar dal/Tur dal` |
| `variety` | keyword | `Arhar Dal(Tur)` |
| `grade` | keyword | `Local` / `Medium` / `FAQ` |
| `arrival_date` | date (`DD/MM/YYYY`) | `13/10/2026` |
| `min_price` / `max_price` / `modal_price` | double | `…` / `…` / `11800` |

- **Sample (13/10/2026):** Bengaluru APMC, tur dal, **modal ₹11,800 / quintal ≈ ₹118/kg** wholesale.
- **Coverage (13/10/2026):** Karnataka 595 records (all returned with `limit=1000`); all-India about 16,954 (needs `offset` pagination). Top Karnataka markets: Davangere 48, Binny Mill (FF&V) Bengaluru 46, Bengaluru APMC 40, Kalaburagi 38, Mysuru 26.
- **Bengaluru markets:** `Bengaluru APMC` (grains, pulses, spices), `Binny Mill (FF&V) Bengaluru APMC` (fruit and vegetables), plus `Ramanagara APMC` (district now "Bengaluru South") and `Channapatna APMC`.
- **Data quality findings:**
  - Several varieties per commodity per market (e.g. 4 onion rows). **Unique key ≈ (arrival_date, market, commodity, variety, grade).**
  - Synonym-packed names (`Red gram split/Arhar dal/Tur dal`) and typos (`Soyabeen`, `Pomogranate`, `Cucumbar`, `Bangalore-Samall`). Canonical names needed.
  - Contradictory labels (`Avare Dal` with variety `Avare (Whole)`).
  - **Units vary by commodity:** most are ₹/quintal, but coconut (`25000`) is likely per 1,000 nuts. A unit is needed per commodity (verify in the catalogue info).
  - District renamed (Ramanagara → Bengaluru South), so location names change over time.
  - Display names are spreadsheet-encoded (`Min_x0020_Price`). Use the field `id`s.
- **Still to verify:** GODL licence terms (quote them); units per commodity; whether past `arrival_date` values can be queried (history) or it's always today's snapshot; the API rate limit and maximum `limit`.
