# Stock Analyst Agent

## Project Goal

Turn a portfolio watchlist (CSV/Excel of tickers) into a repeatable, multi-agent
equity research pipeline that produces, for each stock: actual technical /
fundamental / sentiment values, a 0–10 score per parameter, a weighted overall
score, a BUY / HOLD / SELL / NO ACTION decision, cited sources, and a decision
trail across runs (review date + previous decision).

This is directional research support, not investment advice. Scores and
decisions are inputs to human judgment, not a replacement for it — especially
since default sourcing is live web search, not a licensed market-data feed.

## Folder Structure

```
Stock Analyst Agent/
├── CLAUDE.md                      # this file — project spec, single source of truth
├── agents/
│   ├── agent_1_framework.md       # defines parameters, rubric, weights, thresholds
│   ├── agent_2_technical.md       # researches technical parameters
│   ├── agent_3_fundamental.md     # researches fundamental parameters
│   ├── agent_4_sentiment.md       # researches sentiment/news parameters
│   └── agent_5_decision.md        # aggregates, scores, decides, writes Excel
├── input/
│   └── <portfolio file>.xlsx|csv  # user-supplied list of tickers (not modified by agents)
├── framework/
│   └── framework.json             # Agent 1's output — the contract all other agents read
├── research/
│   ├── technical.json             # Agent 2's output
│   ├── fundamental.json           # Agent 3's output
│   └── sentiment.json             # Agent 4's output
└── output/
    └── Stock Analysis Report.xlsx # Agent 5's output — overwritten/updated each run
```

## Input File Format

Accepts `.xlsx` or `.csv`. Required columns (case-insensitive, header row
detected automatically — the header row is the first row containing a
recognizable "Ticker" column):

| Column | Required | Notes |
|---|---|---|
| Ticker | Yes | Exchange ticker symbol, e.g. `NVDA` |
| Company | No | Full name; looked up if missing |
| State | No | Current position state, e.g. `No Position` / `Long` / `Short`. Defaults to `No Position` if absent. Drives HOLD vs NO ACTION and SELL vs NO ACTION logic in Agent 5. |

Any additional columns in the input file are preserved and passed through
unused. Agent 1 is responsible for locating the ticker list regardless of
minor layout noise (title rows, blank rows) above the header.

## Workflow

1. **Agent 1 (Framework)** reads the input file, extracts the ticker list,
   and defines the full analysis framework → writes `framework/framework.json`.
2. **Agents 2, 3, 4 (Technical / Fundamental / Sentiment)** run **in
   parallel**. Each reads `framework.json` for its parameter list and rubric,
   researches every stock, and writes actual values + score/10 + sources to
   its own file in `research/`.
3. **Agent 5 (Decision)** reads all three research files plus
   `framework.json`, computes category and overall scores, applies decision
   logic, reads the *existing* `output/Stock Analysis Report.xlsx` (if
   present) to carry forward the prior decision, and writes the updated
   Excel report.

Re-running the pipeline (e.g. weekly) re-executes all five steps and updates
`output/Stock Analysis Report.xlsx` in place, preserving history via the
Review Date / Previous Decision columns.

## Analysis Framework (defaults — Agent 1 may tune per input, all values live in `framework.json`)

| Category | Default weight | Default parameters (3–5) |
|---|---|---|
| Technical | 30% | 50/200-day SMA trend, RSI(14), MACD signal, volume trend, position in 52-week range |
| Fundamental | 40% | P/E vs. sector/historical, revenue growth YoY, profit margin trend, debt-to-equity, FCF/EPS growth |
| Sentiment | 30% | Analyst consensus rating & price target, recent news sentiment, short interest, insider activity |

Weights are fundamentals-tilted by default on the assumption of a multi-week+
hold horizon — this is a judgment call, not a fact, and should be revisited if
the actual horizon is shorter (e.g. swing trading would argue for weighting
technical higher).

## Scoring Logic

1. **Parameter score (1–10):** each research agent scores every parameter
   against the rubric Agent 1 defines in `framework.json` (e.g. RSI < 30 →
   8–10 [oversold, bullish], 30–70 → 5–7 [neutral], > 70 → 2–4 [overbought,
   bearish]). Rubrics must be directional and explicit — no unscored "N/A"
   without a documented reason (agent could not find data).
2. **Category score:** simple average of that category's parameter scores.
3. **Overall score:** weighted average of the three category scores using
   the weights in `framework.json`.
4. **Decision (defaults — thresholds configurable in `framework.json`):**

   | Overall score | If `State` = position held | If `State` = No Position |
   |---|---|---|
   | ≥ 7.0 | BUY (add) | BUY |
   | 4.5 – 6.9 | HOLD | NO ACTION |
   | < 4.5 | SELL | NO ACTION |

   Thresholds (7.0 / 4.5) are defaults and must be read from
   `framework.json`, not hard-coded, so the user can tighten or loosen them
   per risk appetite.

## Outputs

`output/Stock Analysis Report.xlsx`, containing:

- **Analysis** sheet (one row per stock):
  - Table 1 — actual values: every technical, fundamental, and sentiment
    parameter's raw value (e.g. RSI = 62, P/E = 34.2x).
  - Table 2 — parameter scores: the same parameters as a 0–10 score.
  - Overall Score (weighted).
  - Decision (BUY / HOLD / SELL / NO ACTION).
  - Review Date (date this run was executed).
  - Previous Decision (decision from the prior run, carried forward before
    being overwritten; blank on first run).
- **Sources** sheet: Ticker × Agent → list of sources cited, so every
  value/score is traceable back to where it came from.

## Agent Roles

| Agent | File | Responsibility | Runs |
|---|---|---|---|
| 1 | `agent_1_framework.md` | Read input, define parameters/rubric/weights/thresholds | First, alone |
| 2 | `agent_2_technical.md` | Research technical parameters, all stocks | Parallel with 3, 4 |
| 3 | `agent_3_fundamental.md` | Research fundamental parameters, all stocks | Parallel with 2, 4 |
| 4 | `agent_4_sentiment.md` | Research sentiment/news parameters, all stocks | Parallel with 2, 3 |
| 5 | `agent_5_decision.md` | Aggregate, score, decide, write Excel | Last, after 2–4 complete |

## Research Method

Default: **live web search** for all values and citations (Agents 2, 3, 4
each list sources used per stock, per parameter, in the Sources sheet).

Alternative: a market/data API (e.g. Alpha Vantage, Financial Modeling Prep,
Polygon.io) can be substituted for technical/fundamental values — sentiment
research still defaults to web search since news/sentiment isn't well served
by most market-data APIs. If switching to an API, the required provider name
and API key must be supplied before Agents 2/3 can use it; record the chosen
provider in `framework.json` under `data_source`.

## Configuration Notes

Everything tunable (parameters, rubric bands, category weights, decision
thresholds, data source) lives in `framework/framework.json`, not hard-coded
in agent prompts, so the pipeline can be re-tuned without editing agent
files.
