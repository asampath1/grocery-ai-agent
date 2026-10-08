# Decision Log

Each entry records a significant decision: context, options, choice, and consequences.
Format: lightweight ADR (Architecture Decision Record).

---

## DEC-001 — Load the coach prompt via CLAUDE.md

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** Claude Code automatically loads `CLAUDE.md` at session start, but not arbitrary files like `coach-prompt.md`. Without it, the coaching rules would be missing unless pasted in every session.
- **Options:**
  1. Paste the prompt manually each session. Easy to forget.
  2. Rename `coach-prompt.md` to `CLAUDE.md`. Loses the descriptive name.
  3. Keep a small `CLAUDE.md` that imports `@coach-prompt.md`. ✅
- **Decision:** Option 3.
- **Consequences:** `CLAUDE.md` is added to the repo structure. Coaching rules apply in every session automatically.

---

## DEC-002 — Revise phase order

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** The original roadmap had no data-acquisition phase even though real integrations are Learning Priority 1. It also put the Frontend MVP before the Claude integration, even though the MVP needs Claude reasoning and the frontend is Priority 5.
- **Changes:**
  - Added **Phase 3: Data Source Investigation**, placed before Database Design so the schema reflects real data.
  - Added **Phase 5: Data Ingestion MVP**.
  - Merged "Claude API Integration" and "Natural Language Querying" into one phase. NL querying through tool calling *is* the first meaningful Claude integration.
  - Moved **Frontend** to after Claude integration and price comparison.
  - The total went from 12 to 13 phases.
- **Trade-off:** There's no visual demo until Phase 10. This is acceptable because the learning priorities rank the pipeline and AI work above the UI, and progress can be demoed from the terminal or Swagger UI.

---

## DEC-003 — Data access policy (scraping vs. official APIs)

- **Date:** 2026-10-08
- **Status:** ⏳ **Open**, to be decided during Phase 3
- **Context:** Major Indian grocery platforms (Blinkit, Zepto, Swiggy Instamart, BigBasket, JioMart, Amazon Fresh) don't appear to offer public pricing APIs, and their terms of service generally restrict automated access. Prices are pincode-specific.
- **Options to evaluate:**
  1. Official or partner/affiliate APIs, where they exist
  2. Calling the internal JSON endpoints the platform's own web app uses
  3. HTML scraping
  4. Manual or semi-manual data capture
  5. Mock data, as a last resort only
- **Decision criteria:** legality/ToS, stability, learning value, effort.

---

## DEC-004 — Location scope

- **Date:** 2026-10-08
- **Status:** Accepted
- **Decision:** All price lookups target **pincode 560016 (Bangalore)**.
- **Reason:** Quick-commerce prices and stock depend on pincode. Fixing one location keeps data comparable and the scope small. The pincode is stored per price record so more locations can be added later.

---

## DEC-005 — Docker runtime on macOS: OrbStack

- **Date:** 2026-10-08
- **Status:** Accepted
- **Context:** Docker on macOS needs a Linux VM. The options were Docker Desktop, OrbStack, and Colima.
- **Decision:** OrbStack.
- **Reasons:** Lightest on resources, native Apple Silicon support, very little setup, free for personal use.
- **Trade-offs:** Closed source and from a small company. If that becomes a problem, Colima (MIT licence) is a drop-in replacement, because containers and Compose files don't depend on the runtime.
- **Note:** OrbStack starts with a 30-day Pro trial. After it ends, choose the free personal-use tier. No Pro features are needed.
