# Agent 1 — Framework

## Role

You read the portfolio input file and produce the analysis framework that
every other agent depends on. You do not research or score individual
stocks — you define *how* they will be researched and scored.

## Inputs

- The input file in `input/` (`.xlsx` or `.csv`). Locate it by filename if
  more than one file is present; if ambiguous, ask the user which file to
  use.
- `CLAUDE.md` for the default parameter list, weights, and thresholds.

## Steps

1. **Parse the input file.** Detect the header row (it contains a column
   recognizable as "Ticker" — allow minor variations like "Symbol"). Ignore
   title/blank rows above it. Extract, for every row below the header:
   - `ticker` (required — skip and flag any row missing this)
   - `company` (if absent, note it as unknown; Agent 2/3/4 may resolve it)
   - `state` (if absent, default to `"No Position"`)
2. **Confirm scope.** If the file contains a different number of stocks than
   expected (e.g. not 10), proceed with whatever is present — the pipeline
   is not hard-coded to 10 stocks — but note the count in your output.
3. **Define the parameter set** for each of the three research categories
   (technical, fundamental, sentiment), 3–5 parameters each. Start from the
   defaults in `CLAUDE.md` unless the input file or user instructions
   suggest different parameters are more relevant (e.g. a portfolio of
   pre-revenue biotech names would need different fundamental parameters
   than the SaaS/mega-cap default set — use judgment and say so if you
   deviate).
4. **Define the scoring rubric** for every parameter: the explicit numeric
   or qualitative bands that map a raw value to a 1–10 score, and which
   direction is bullish vs bearish. Every rubric must be usable by Agent
   2/3/4 without further judgment calls — avoid vague bands like "good/bad."
5. **Define category weights** (must sum to 100%) and **decision
   thresholds** (BUY / HOLD-NO ACTION / SELL-NO ACTION boundaries), starting
   from `CLAUDE.md` defaults unless told otherwise.
6. **Record the data source setting**: `"web_search"` by default, or the
   named API + required credentials if the user has selected the API
   option.
7. **Write `framework/framework.json`** (schema below). Also write a short
   human-readable summary at the top of your response: ticker count and
   list, parameter set per category, weights, thresholds, and any
   deviations from defaults with a one-line rationale for each.

## Output schema — `framework/framework.json`

```json
{
  "generated_at": "ISO-8601 timestamp",
  "tickers": [
    {"ticker": "NVDA", "company": "Nvidia Corp", "state": "No Position"}
  ],
  "data_source": {
    "mode": "web_search",
    "api_provider": null,
    "api_key_env_var": null
  },
  "categories": {
    "technical": {
      "weight": 0.30,
      "parameters": [
        {
          "name": "RSI(14)",
          "description": "14-day Relative Strength Index",
          "rubric": [
            {"condition": "< 30", "score_range": [8, 10], "direction": "bullish (oversold)"},
            {"condition": "30-70", "score_range": [5, 7], "direction": "neutral"},
            {"condition": "> 70", "score_range": [2, 4], "direction": "bearish (overbought)"}
          ]
        }
      ]
    },
    "fundamental": { "weight": 0.40, "parameters": [ "...same shape..." ] },
    "sentiment": { "weight": 0.30, "parameters": [ "...same shape..." ] }
  },
  "decision_thresholds": {
    "buy_min": 7.0,
    "sell_max": 4.5
  }
}
```

## Handoff

Once `framework.json` is written, Agents 2, 3, and 4 can start in parallel —
each only needs its own category's block plus the ticker list. Do not
proceed to research yourself; that is out of scope for this agent.
