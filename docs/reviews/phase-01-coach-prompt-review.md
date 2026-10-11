# Review 01: Coach Prompt & Project Plan

- **Date:** 2026-10-08
- **Phase:** 1 — Repository Setup
- **Reviewed:** `coach-prompt.md` (first version)
- **Decisions produced:** DEC-001 to DEC-006

## Summary

Before any work started, the coach read the original prompt and raised 12 questions. Two of them exposed real design flaws in the plan: there was no phase for getting data, and the frontend came before Claude. One challenged whether Claude was needed for the MVP question at all. The rest calibrated the coaching to my skills, time, and preferences.

## My answers (word for word)

> 1. I agree and its ok. suggest you to do what is required as the best.2. Create the folder structure ( i have not created it).for Q3and Q4. I just put what i had i mind, but now you know my intentions, so accordingly make modifications wherever necessay but do guide me. Q5.my pincode 560016, Bangalore. 6. Agree, 7 Python/SQL/Docker/Linux erver/git are comfortable. Javascript/React-totally new.8 an hour/day is my target unless there are other priorities. , so lets take this as 8hrs/week. that way, i can adust 9. Yes - Github , Hostinger premium account ( but i can upgrade/ no docker in mac. need to check python/node n my mac.10 For docs, I want you to write. 11. use shorter version.12.cal day

---

## Item 1: How does the coach prompt get loaded?

- **Coach challenge:** Claude Code only auto-loads `CLAUDE.md`. Without it, every session would start without the coaching rules.
- **My answer:** "I agree and its ok. suggest you to do what is required as the best."
- **Outcome:** A small `CLAUDE.md` that imports `@coach-prompt.md`. → [DEC-001](../decision-log.md#dec-001--load-the-coach-prompt-via-claudemd)

## Item 2: Data acquisition was missing from the roadmap

- **Original plan:** 12 phases, none about getting data, even though real integrations were Learning Priority 1.
- **Coach challenge:** The top priority had no phase. The database also shouldn't be designed before we know what real sources return.
- **My answer:** "I just put what i had i mind, but now you know my intentions, so accordingly make modifications wherever necessay but do guide me."
- **Outcome:** Added Phase 3 (Data Source Investigation, *before* Database Design) and Phase 5 (Data Ingestion MVP). → [DEC-002](../decision-log.md#dec-002--revise-phase-order)

## Item 3: The frontend came before Claude

- **Original plan:** Phase 5 Frontend → Phase 6 Claude, but the MVP needs Claude and frontend polish was Priority 5.
- **Coach challenge:** Swap them. A plain endpoint or CLI can prove the MVP.
- **My answer:** (same as Item 2)
- **Outcome:** Claude moved to Phase 7, the frontend to Phase 10. The trade-off, no visual demo until Phase 10, was accepted. → [DEC-002](../decision-log.md#dec-002--revise-phase-order)

## Item 4: Is scraping acceptable? Which location?

- **Coach challenge:** None of the major Indian grocery platforms appears to have a public pricing API, their terms usually restrict scraping, and prices depend on pincode.
- **My answer:** "my pincode 560016, Bangalore." (Scraping comfort not yet answered.)
- **Outcome:** Location fixed → [DEC-004](../decision-log.md#dec-004--location-scope). Scraping policy left open → [DEC-003](../decision-log.md#dec-003--data-access-policy-scraping-vs-official-apis), to be decided in Phase 3.

## Item 5: Does the MVP even need Claude?

- **Coach challenge:** "Where is X cheapest?" is just `ORDER BY price LIMIT 1`. Claude has to earn its place through understanding questions (tool calling) or matching products across stores.
- **My answer:** "Agree"
- **Outcome:** Added to the coach prompt's MVP section and the project vision. This shaped DEC-008 and DEC-010 later.

## Item 6: Calibration (skills, time, working style)

| Question | My answer | What changed |
|---|---|---|
| Skill level | "Python/SQL/Docker/Linux erver/git are comfortable. Javascript/React-totally new." | Explanations go deep on design and assume backend basics. JS/React will be taught from fundamentals. |
| Time | "an hour/day … lets take this as 8hrs/week" | Milestones sized for 1-hour sessions. Roadmap estimated at about 20–25 weeks. |
| Accounts | "Github , Hostinger premium account ( but i can upgrade/ no docker in mac" | Flagged: Hostinger "Premium" is likely shared hosting, which can't run Docker. Check before Phase 12. |
| Who writes docs | "For docs, I want you to write." | Coach writes docs; I write code. Added to "Division of Work". |
| Task format | "use shorter version." | Short format for small tasks, full format for milestones. |
| Journal naming | "cal day" | One journal file per calendar day worked. |

## Item 7: Docker runtime

- **Coach challenge:** Docker Desktop, OrbStack, or Colima? Compared on cost, resource use, and risk.
- **My answer:** Installed OrbStack.
- **Outcome:** → [DEC-005](../decision-log.md#dec-005--docker-runtime-on-macos-orbstack)

## Item 8: My question: "What is Projects in GitHub? Are we not going to use it?"

- **Context:** The coach hadn't planned to use GitHub Projects, to avoid tracking progress in two places.
- **Coach response:** Use it, but give each tool its own job: Issues for tasks, `PROJECT_STATUS.md` for the phase summary.
- **My answer:** "yes, add it to the coach prompt"
- **Outcome:** → [DEC-006](../decision-log.md#dec-006--task-tracking-with-github-issues--projects)

---

## What improved

- **Before:** A 12-phase plan that started with the UI and had no data phase, and an unclear role for Claude.
- **After:** A 13-phase plan that puts the riskiest part (data access) first, makes Claude justify its place, and has coaching calibrated to me.
- **My main lesson:** Ask what each component does that plain code can't, and tackle the biggest unknown first.
