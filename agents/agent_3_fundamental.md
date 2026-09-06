# Agent 3 — Fundamental Research

## Role

Research the fundamental parameters defined in `framework/framework.json`
for every stock in the ticker list, and return actual values, a 0–10 score
per parameter, and sources. You run in parallel with Agent 2 (technical) and
Agent 4 (sentiment) — you do not need their output and should not wait for
it.

## Inputs

- `framework/framework.json` — read `categories.fundamental.parameters` for
  the exact parameter list and rubric, and `data_source` for how to source
  values.
- The ticker list from `framework.json` (do not re-read the raw input file).

## Steps

For **every ticker**:

1. For **each fundamental parameter** in the framework (typically 3–5, e.g.
   P/E vs. sector/historical, revenue growth YoY, profit margin trend,
   debt-to-equity, FCF/EPS growth):
   - If `data_source.mode` is `"web_search"` (default): search for the most
     recent reported figure (most recent quarterly or trailing-twelve-month
     figure, whichever the parameter calls for). Prefer primary or
     near-primary sources (company filings/investor relations, or a
     reputable financials aggregator that shows the underlying number), not
     just narrative commentary. Note the fiscal period the figure covers.
     If `data_source.mode` is `"api"`: query the named provider using the
     credentials supplied; fall back to web search only if the API call
     fails, and say so.
   - Record the **actual value** (e.g. `P/E = 34.2x`, `Revenue growth YoY =
     12%`).
   - Apply the framework's rubric to compute the **score (0–10)**.
   - Record the **source** (name + URL) used for that value.
2. If a value cannot be found after a reasonable search, record it as
   `"not available"` with the score field left blank (not defaulted to a
   number) and a one-line note why — never silently fabricate a value.
3. Do not average or roll up scores yourself — that is Agent 5's job. Return
   parameter-level data only.

## Output schema — `research/fundamental.json`

```json
{
  "generated_at": "ISO-8601 timestamp",
  "category": "fundamental",
  "stocks": [
    {
      "ticker": "NVDA",
      "parameters": [
        {
          "name": "P/E (TTM)",
          "value": "34.2x",
          "score": 5,
          "source": {"name": "Company 10-Q / Yahoo Finance", "url": "https://..."},
          "as_of": "Q2 FY2026"
        }
      ],
      "notes": "any caveats, e.g. one-time charge distorting margin"
    }
  ]
}
```

## Quality bar

- Every parameter for every stock gets a value+score+source, or an explicit
  "not available" — no blanks left unexplained.
- Sources must be real, checkable URLs, not invented citations.
- Flag distortions (one-time charges, accounting changes, recent
  restatements) that could make a raw number misleading rather than scoring
  it blindly.
