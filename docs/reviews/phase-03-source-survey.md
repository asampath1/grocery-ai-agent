# Review 04: Grocery Platform Survey (pincode 560016)

- **Date:** 2026-10-13
- **Phase:** 3 — Data Source Investigation
- **Task:** [#6](https://github.com/asampath1/grocery-ai-agent/issues/6): Survey grocery platforms for 560016
- **Platforms:** Blinkit (full technical inspection), BigBasket, JioMart (terms of service), Amazon (official API)
- **Reference:** [docs/data-sources.md](../data-sources.md)
- **Outcome:** ❌ All four commercial routes are blocked: three by terms of service, one by eligibility. Feeds into DEC-003 (still open).

---

## Part 1: Blinkit technical inspection (DevTools)

### What I found, step by step

| Step | My observation (word for word) | Coach feedback |
|---|---|---|
| 1. First look at the Network tab | (screenshot of 68 requests) | Most requests are noise: `collect` (Google Analytics), `937732702/?random` (Ads), `events` (tracking). Filter them out with `-collect -events -google -random`. The **Initiator** column shows `sentryvendor.js` for everything because Sentry *wraps* `fetch`. |
| 2. Found the price | "i could price key in the json" | `price: 88, mrp: 118` for `product_id "305650"`, i.e. **500 g**. The coach's guess that the two IDs were group and variant was **wrong**; they're separate products for different pack sizes. |
| 3. Second ID | (screenshot) | `39493` = **1 kg, ₹165, MRP ₹224**. Same fields with **different types and empty-value styles** across responses (`type_id` 14489 vs "14489"; `""` vs `null` vs `"#-NA"`). |
| 4. Pack size | "found the unit." | In a `cart_item` block: `unit: "1 kg"`, `group_id: 2214592`, consistent fields. **Parse from data the app depends on to work (cart), not from tracking data.** |
| 5. robots.txt | (screenshot) | Product pages (`/prn/…/prid/…`) are **allowed**; search page `/s/*` is disallowed; the sitemap is explicitly allowed. The API path `/v1/layout/search` isn't listed. |
| 6. API host and SSR | "request url has blinkit.com https://blinkit.com/v1/layout/search?…" | Same host, `layout` in the path, so the server sends a **screen layout**. The product page HTML also embeds the price as JSON (`"₹165/kg"`). |
| 7. ToS | "from ToS, i understand it is probhited." | ✅ Correct. Prices are named explicitly. **The ToS outweighs robots.txt.** |

### Blinkit technical summary

| Aspect | Finding |
|---|---|
| Location | Map pin / coordinates (Plus Code), not pincode. `visibility?latitude=…` → `merchant_id: 31434` (dark store) |
| Data delivery | Client-side JSON (`/v1/layout/search`, product requests by ID) **and** JSON embedded in SSR HTML |
| Key fields | `price`, `mrp`, `unit` ("1 kg"), `inventory`, `state` ("available"), `brand`, `group_id`, `merchant_id`, `price_per_unit` ("₹165/kg") |
| Data quality | Mixed types for the same field, 3 kinds of empty value, layout-driven responses (fragile) |
| Friction | No login or anti-bot challenge seen while browsing |
| Technical feasibility | 4/5 |
| **Permission** | **1/5: ToS prohibits** |

---

## Part 2: Terms of service

| Platform | Key clause (quoted) | Covers manual capture? | Covers our own DB? |
|---|---|---|---|
| **Blinkit** | "…any collection, use or derivative use of … product listings, descriptions, or **prices**; … or any use of data mining, robots, or similar data gathering and extraction tools." | Not explicit | Not explicit |
| **BigBasket** (10.9) | "You shall not use any 'deep-link', 'page-scrape', 'robot', 'spider' or other automatic device, program, algorithm or methodology, **or any similar or equivalent manual process**, to access, acquire, copy **or monitor** any portion of the Platform…" | ❌ **Yes** | Not explicit |
| **JioMart** (5.2) | "(b) any collection and use of product listings, description, or **prices**, … (e) any use of data mining, robots… to extract (**whether once or many times**) … (e) **creating and/ or publishing your own database that features parts of JioMart**." | ❌ Yes (collection of prices) | ❌ **Yes** |

My words: BigBasket: "bigbasket done. same case." JioMart: "same with JioMart Tos."

**Coach note:** The JioMart clause on "your own database" affects the *architecture*, not just the way data is fetched: storing their prices in our PostgreSQL is itself excluded.

---

## Part 3: Amazon official API

| Aspect | Finding |
|---|---|
| Route | Amazon Associates → **Creators API** (includes PA API), `affiliate-program.amazon.in/creatorsapi` |
| Eligibility | Approved Associates account **plus ≥10 qualifying sales in the past 30 days** |
| My status | Not an Associate (page shows "An error occurred") |
| Auth | Up to 2 applications × 2 credential sets, intended for **rotation** |
| Response shape | Designed contract: `offersV2.listings[].price.money.{amount,currency}`, `savingBasis`, `pricePerUnit`, `availability.type` |
| **Blocker** | **Chicken-and-egg**: you need sales to get the API, and the app to get sales |
| Open questions | Pincode-specific pricing? Licence limits on caching and on showing prices next to other retailers? |

---

## What improved (what I now understand)

| Before Task 3.1 | After |
|---|---|
| "Indian grocery platforms probably have no public API" | Verified: none of the 3 has a public API; Amazon's needs affiliate sales |
| Pincode identifies location | Quick-commerce uses **coordinates → dark store (`merchant_id`)**. DEC-004 to be revisited |
| A price is a field | Prices sit in layouts, with mixed types and 3 kinds of empty value. **Parse from functional data (cart)** |
| robots.txt decides | **ToS outweighs robots.txt**: robots.txt is for search engines, the ToS is the agreement you accept |
| Inspect first, check terms later | **Check terms first**: 2 minutes, and it decides the outcome |
| Scraping = automation | BigBasket prohibits *manual* monitoring too; JioMart prohibits *your own database* of its data |

**My main lesson:** "Can I get the data?" has three layers: technical, robots.txt, and contractual. The contractual one decides, and it's the cheapest to check, so check it first.
