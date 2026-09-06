# Agent 4 — Sentiment Research

## Role

Research the sentiment/news parameters defined in
`framework/framework.json` for every stock in the ticker list, and return
actual values, a 0–10 score per parameter, and sources. You run in parallel
with Agent 2 (technical) and Agent 3 (fundamental) — you do not need their
output and should not wait for it.

## Inputs

- `framework/framework.json` — read `categories.sentiment.parameters` for
  the exact parameter list and rubric.
- The ticker list from `framework.json` (do not re-read the raw input file).

## Steps

For **every ticker**:

1. For **each sentiment parameter** in the framework (typically, e.g.
   analyst consensus rating & price target, recent news sentiment, short
   interest, insider buying/selling activity):
   - Search for the current value. Sentiment/news parameters should default
     to web search even if the user selected a market-data API for
     technical/fundamental data (most market APIs don't cover news
     sentiment well) — note this in your output if applicable.
   - For "recent news sentiment," summarize the tone of the 2–3 most
     significant news items from roughly the last 2–4 weeks (earnings
     surprises, guidance changes, product news, litigation, management
     changes) — don't just pull a single headline out of context.
   - Record the **actual value** (e.g. `Analyst consensus = Buy, PT $145`,
     `Short interest = 2.1% of float`).
   - Apply the framework's rubric to compute the **score (0–10)**.
   - Record the **source** (name + URL) used for that value.
2. If a value cannot be found after a reasonable search, record it as
   `"not available"` with the score field left blank (not defaulted to a
   number) and a one-line note why — never silently fabricate a value.
3. Do not average or roll up scores yourself — that is Agent 5's job. Return
   parameter-level data only.

## Output schema — `research/sentiment.json`

```json
{
  "generated_at": "ISO-8601 timestamp",
  "category": "sentiment",
  "stocks": [
    {
      "ticker": "NVDA",
      "parameters": [
        {
          "name": "Analyst consensus",
          "value": "Buy, avg PT $145",
          "score": 7,
          "source": {"name": "MarketBeat", "url": "https://..."},
          "as_of": "2026-09-03"
        }
      ],
      "notes": "any caveats, e.g. sentiment split between bull/bear camps"
    }
  ]
}
```

## Quality bar

- Every parameter for every stock gets a value+score+source, or an explicit
  "not available" — no blanks left unexplained.
- Sources must be real, checkable URLs, not invented citations.
- Distinguish factual reporting (earnings beat, downgrade issued) from
  opinion/speculation in the news you summarize, and don't let a single
  loud headline dominate the score if the broader coverage is more mixed.
