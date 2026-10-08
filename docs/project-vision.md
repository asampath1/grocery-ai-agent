# Project Vision

## Problem

Grocery prices for the same product differ across Indian platforms (quick-commerce apps, e-commerce, and supermarket chains) and change often. Comparing them by hand means opening several apps, and the products often have different names on each.

## What we're building

An AI-powered system that:

1. **Collects** live prices for real products from real platforms (pincode 560016, Bangalore).
2. **Stores** them in PostgreSQL with history.
3. **Matches** the same product across stores even when names differ.
4. **Answers** natural-language questions using Claude with tool calling over the stored data.

## First milestone question

> "Where is Tata Sampann Toor Dal 1kg cheapest?"

## Where Claude actually adds value

A plain SQL `ORDER BY price` can find the cheapest price. Claude is used where plain code struggles:

- **Understanding questions:** turning free-form questions into structured queries (tool calls).
- **Product matching:** recognising that "Tata Sampann Unpolished Toor Dal 1 kg" and "Tata Sampann Arhar Dal (1kg)" are the same product.
- **Reasoning and explanation:** basket optimisation trade-offs, savings summaries, and trends.

## Long-term questions

- "Which store has the lowest total basket cost?"
- "How much can I save this month?"
- "Create a grocery basket under ₹5000."
- "Which products are cheaper on JioMart than on Amazon?"

## Learning goals (primary)

Real-world data integration → data pipelines → AI reasoning over structured data → deployment → frontend.

## Non-goals (for now)

- Agent frameworks (LangChain, CrewAI, AutoGen)
- Supporting many cities or pincodes
- Real-time price alerts or notifications
- A mobile app
