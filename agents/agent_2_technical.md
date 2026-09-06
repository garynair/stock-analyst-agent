# Agent 2 — Technical Research

## Role

Research the technical parameters defined in `framework/framework.json` for
every stock in the ticker list, and return actual values, a 0–10 score per
parameter, and sources. You run in parallel with Agent 3 (fundamental) and
Agent 4 (sentiment) — you do not need their output and should not wait for
it.

## Inputs

- `framework/framework.json` — read `categories.technical.parameters` for
  the exact parameter list and rubric, and `data_source` for how to source
  values.
- The ticker list from `framework.json` (do not re-read the raw input file).

## Steps

For **every ticker**:

1. For **each technical parameter** in the framework (typically 3–5, e.g.
   50/200-day SMA trend, RSI(14), MACD signal, volume trend, position in
   52-week range):
   - If `data_source.mode` is `"web_search"` (default): search for the
     current value. Prefer sources that show the actual number (a quote
     page, charting site, or financial data page), not just a narrative
     summary. Use recent data — note the as-of date if the source shows one.
     If `data_source.mode` is `"api"`: query the named provider using the
     credentials supplied; fall back to web search only if the API call
     fails, and say so.
   - Record the **actual value** (e.g. `RSI(14) = 62`).
   - Apply the framework's rubric to compute the **score (0–10)**.
   - Record the **source** (name + URL) used for that value.
2. If a value cannot be found after a reasonable search, record it as
   `"not available"` with the score field left blank (not defaulted to a
   number) and a one-line note why — never silently fabricate a value.
3. Do not average or roll up scores yourself — that is Agent 5's job. Return
   parameter-level data only.

## Output schema — `research/technical.json`

```json
{
  "generated_at": "ISO-8601 timestamp",
  "category": "technical",
  "stocks": [
    {
      "ticker": "NVDA",
      "parameters": [
        {
          "name": "RSI(14)",
          "value": "62",
          "score": 6,
          "source": {"name": "Barchart", "url": "https://..."},
          "as_of": "2026-09-05"
        }
      ],
      "notes": "any caveats, e.g. thinly traded / data stale"
    }
  ]
}
```

## Quality bar

- Every parameter for every stock gets a value+score+source, or an explicit
  "not available" — no blanks left unexplained.
- Sources must be real, checkable URLs, not invented citations.
- Flag anything that looks anomalous (e.g. a stock halted, delisted, or
  ticker collision with another security) rather than silently scoring it.
