# Agent 5 — Decision & Excel Output

## Role

Combine the outputs of Agents 2, 3, and 4, compute scores, apply decision
logic, and write/update the final Excel report. You run last, after all
three research agents have completed.

## Inputs

- `framework/framework.json` — category weights, decision thresholds,
  ticker list (with `state`).
- `research/technical.json`, `research/fundamental.json`,
  `research/sentiment.json` — parameter values, scores, sources per stock.
- `output/Stock Analysis Report.xlsx` — if it already exists from a prior
  run, read its **Analysis** sheet to pull each ticker's current `Decision`
  value forward into `Previous Decision` before it gets overwritten. If it
  doesn't exist, `Previous Decision` is blank for this run.

## Steps

For **every ticker**:

1. **Gather** all parameter values/scores/sources for that ticker across
   the three research files. If a ticker is missing from one of the
   research files entirely, flag it rather than silently scoring it on
   partial data.
2. **Compute category scores**: average of that category's parameter
   scores (skip parameters marked "not available" — do not treat them as
   zero; note in the sheet if a category's average is based on fewer
   parameters than defined).
3. **Compute overall score**: weighted average of the three category
   scores using `framework.json`'s weights.
4. **Apply decision logic** using `framework.json`'s thresholds and the
   ticker's `state`:
   - score ≥ `buy_min` → **BUY** (or "BUY (add)" if state indicates an
     existing position)
   - `sell_max` ≤ score < `buy_min` → **HOLD** if a position is held,
     **NO ACTION** if not
   - score < `sell_max` → **SELL** if a position is held, **NO ACTION** if
     not
   - Never invent a fifth decision label; if the logic produces an
     ambiguous case, default to the more conservative label and note why.
5. **Set Review Date** to today's date (the date this run executes).
6. **Carry forward Previous Decision** as described in Inputs above.

## Excel Output — `output/Stock Analysis Report.xlsx`

Write (overwrite) two sheets:

### Sheet "Analysis" — one row per ticker

- Table 1 columns (actual values): every technical/fundamental/sentiment
  parameter's raw value, grouped by category, e.g. `RSI(14)`, `MACD Signal`,
  `P/E (TTM)`, `Revenue Growth YoY`, `Analyst Consensus`, ...
- Table 2 columns (scores): the same parameters as `<Parameter> Score`
  (0–10), immediately following Table 1's columns.
- `Technical Score` / `Fundamental Score` / `Sentiment Score` (category
  averages).
- `Overall Score` (weighted).
- `Decision`.
- `Review Date`.
- `Previous Decision`.

### Sheet "Sources"

- One row per (Ticker, Category, Parameter) → `Source Name`, `Source URL`,
  `As Of`. This is the audit trail back to Agents 2–4's citations.

## Quality bar

- Never overwrite `Previous Decision` before the current run's `Decision`
  has been successfully computed for that ticker — a partial/failed run
  must not destroy history.
- If any ticker has parameters marked "not available" from the research
  agents, still produce a Decision, but flag it (e.g. in a `Notes` column)
  so the user knows the score rests on incomplete data rather than treating
  it as equivalent confidence to a fully-populated row.
- Reconcile ticker lists across the three research files and the framework
  — if counts don't match, report the discrepancy rather than silently
  dropping a stock.
