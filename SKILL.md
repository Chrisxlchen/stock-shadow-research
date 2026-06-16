---
name: stock-shadow-research
description: Run a repeatable stock shadow-research report from a ticker by orchestrating Serenity-style alpha, GF-DMA trend health, TAM-Adj-PEG valuation, Bayesian intrinsic-growth valuation, buy-side memo, and Prism three-lens synthesis. Use when the user asks to "run shadow research", "stock shadow research", "use the new stock skills", "analyze a ticker with Serenity/Prism", or wants a report from a skill name plus ticker.
---

# Stock Shadow Research

## Purpose

Produce a sidecar research report for one stock without changing any production
pool, score, order, or sizing decision. This skill is an orchestration layer: it
uses the installed research lenses as frameworks and returns a consistent report.

Treat every output as research analysis, not personalized investment advice.

## Prerequisites

This skill assumes these supporting skills are installed before use:

Required:

- `prism-skill` from `https://github.com/destiny520537work-lab/prism-skill`
- Serenity skills from `https://github.com/haskaomni/serenity-skill`:
  - `serenity-alpha`
  - `gf-dma-health-index`
  - `tam-adj-peg`
  - `bayesian-intrinsic-growth-valuation`
  - `buy-side-equity-research-memo`

Optional:

- UZI-Skill from `https://github.com/wbh604/UZI-Skill` if the user also wants
  to run that separate external deep-analysis tool. This wrapper does not
  require UZI-Skill for its default report.

Install examples:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo destiny520537work-lab/prism-skill --path . --name prism-skill --method git

python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo haskaomni/serenity-skill \
  --path skills/bayesian-intrinsic-growth-valuation \
         skills/buy-side-equity-research-memo \
         skills/gf-dma-health-index \
         skills/serenity-alpha \
         skills/tam-adj-peg \
  --method git
```

Restart Codex after installing supporting skills.

## User Interface

Accept natural language such as:

```text
Run stock-shadow-research for MU.
Run stock-shadow-research for MU ALAB DDOG.
用 stock-shadow-research 分析 MU。
用 stock-shadow-research 分析 MU、ALAB、DDOG。
给 MU 跑 shadow research report。
用 Serenity + Prism 给 MU 出一份旁路研究报告。
```

If the user supplies multiple tickers, run the workflow once per ticker. Treat
comma-separated, whitespace-separated, and Chinese-list punctuation as ticker
separators. If the user supplies only a ticker, infer U.S. listed common stock
unless the symbol is ambiguous. Ask only when ambiguity would materially change
the company.

## Required Data Collection

Before writing the report, collect current data because price, estimates,
macro, and technical state are time-sensitive:

1. Current quote, market cap, forward P/E, trailing P/E, P/S, PEG if available.
2. Latest earnings release, guidance, and investor presentation / prepared remarks.
3. Industry or supply-chain data relevant to the thesis.
4. Technical data: latest price, 20/50/100/200DMA, ATR20, recent returns, and
   moving-average slopes. Prefer local `yfinance` calculation when available.
5. Macro backdrop: U.S. 10Y yield and near-term Fed expectation.

Use primary sources first: company IR, SEC filings, exchange filings, official
guidance, and earnings materials. Use reputable market data sources for quote,
estimates, and technicals. Clearly label stale or unavailable data.

## Framework Order

Run the report in this order:

1. **Data Summary**: timestamp, quote, valuation, latest earnings, guidance,
   technical snapshot, macro snapshot, and key sources.
2. **Serenity Alpha**: demand change -> revenue/profit transmission ->
   verification chain -> downside risks.
3. **GF-DMA Health Index**: score out of 100 using growth match, price-DMA
   divergence, trend parallelism, and estimate revisions.
4. **TAM-Adj-PEG**: forward valuation vs EPS growth, TAM runway, quality factor,
   cyclicality, and valuation support.
5. **Bayesian Intrinsic Growth**: H0-H5 probability table, weighted intrinsic
   growth, market-implied growth, posterior updates, and validation path.
6. **Buy-Side Memo**: thesis, variant perception, valuation/scenarios,
   catalysts, risks, and monitoring dashboard.
7. **Prism Three-Lens Synthesis**: supply-chain bottleneck, macro regime, and
   technical execution. Keep the three views distinct.
8. **Final Shadow Verdict**: what the production system should watch, but do not
   recommend changing production fields unless the user explicitly asks for an
   implementation plan.
9. **Persist Artifacts**: write `rank-history/{YYYY-MM-DD}/{TICKER}/report.md`
   and `rank-history/{YYYY-MM-DD}/{TICKER}/{TICKER}.json` before the final chat
   response.

Do not paste or load the full underlying skill bodies unless needed. The
installed lenses this skill orchestrates are:

- `serenity-alpha`
- `gf-dma-health-index`
- `tam-adj-peg`
- `bayesian-intrinsic-growth-valuation`
- `buy-side-equity-research-memo`
- `prism-skill`

## Output Contract

Default language: Chinese.

Default persistence behavior when working inside the `trade-bot` repo:

1. Create `rank-history/{YYYY-MM-DD}/{TICKER}/` if it does not exist.
2. Write the human-readable Markdown report to
   `rank-history/{YYYY-MM-DD}/{TICKER}/report.md`.
3. Write the machine-readable JSON file to
   `rank-history/{YYYY-MM-DD}/{TICKER}/{TICKER}.json`.
4. Mention both output paths for each ticker in the final chat response.

Use the current local date for `{YYYY-MM-DD}` unless the user provides an
explicit as-of date. Uppercase the ticker in filenames.

Default overwrite behavior: if the same ticker is run again on the same date,
overwrite that ticker's `report.md` and `{TICKER}.json`. If the user explicitly
asks to keep versions, write timestamped files instead:

```text
rank-history/{YYYY-MM-DD}/{TICKER}/report-{HHMMSS}.md
rank-history/{YYYY-MM-DD}/{TICKER}/{TICKER}-{HHMMSS}.json
```

Default shape:

```markdown
**Data Summary**
...

**1. Serenity Alpha**
...

**2. GF-DMA Health Index**
...

**3. TAM-Adj-PEG**
...

**4. Bayesian Intrinsic Growth**
...

**5. Buy-Side Memo**
...

**6. Prism 三人合议**
...

**Final Shadow Verdict**
...

Sources: ...
> 以上为框架推演，不构成投资建议。
```

Keep the final chat answer concise: summarize each ticker's verdict and list the
files written. The full report belongs in
`rank-history/{date}/{ticker}/report.md`.
Read `references/shadow-artifact-schema.md` when writing the JSON or appending
to an existing same-day report.

For the per-ticker JSON, use a compact production-friendly shape:

```json
{
  "ticker": "MU",
  "research_quality": "high",
  "thesis_quality": "high",
  "fundamental_acceleration": "very_high",
  "industry_chain_importance": "high",
  "technical_entry_quality": "poor",
  "valuation_risk": "medium_high",
  "cyclicality_risk": "high",
  "winner_preservation_flag": true,
  "new_entry_bias": "wait_for_pullback",
  "existing_position_bias": "hold_with_system_rules",
  "thesis_break_risk": "medium",
  "thesis_break_triggers": [
    "gross margin drops materially below guide",
    "HBM visibility weakens",
    "hyperscaler capex revisions turn down",
    "DRAM/NAND pricing rolls over"
  ]
}
```

Allowed value style:

- quality fields: `low`, `medium`, `high`
- acceleration / importance fields: `low`, `medium`, `high`, `very_high`
- entry quality: `poor`, `mixed`, `good`, `excellent`
- risk fields: `low`, `medium`, `medium_high`, `high`
- `new_entry_bias`: `avoid`, `wait_for_pullback`, `small_test_only`,
  `acceptable`, `strong`
- `existing_position_bias`: `trim_or_exit`, `hold_with_tight_risk`,
  `hold_with_system_rules`, `add_only_on_confirmation`

## Production Boundary

This skill must not edit:

- `pools/*.json`
- `config/universal_tickers.json`
- production ranking or sizing code
- database state
- order-generation logic

It may suggest future fields for a separate `research_shadow.json` artifact, but
active integration requires an explicit follow-up request and a separate design
or ADR.

## Batch Failure Handling

For multiple tickers, isolate failures per ticker. Do not stop the whole batch
because one ticker has ambiguous identity, missing data, source failures, or
technical-data gaps.

If a ticker fails:

1. Still create `rank-history/{YYYY-MM-DD}/{TICKER}/`.
2. Write `report.md` explaining the failure, missing data, and suggested user
   inputs.
3. Write `{TICKER}.json` with `research_quality: "low"` plus an `error` field
   describing the failure.
4. Continue to the next ticker.

In the final chat response, separate successful tickers from failed tickers.

## Scoring Heuristics

Use transparent scoring, not false precision.

For GF-DMA, use the installed skill's weighting:

```text
HealthScore = 40% GrowthMatch + 25% Divergence + 20% Parallel + 15% Revision
```

For Bayesian growth, use H0-H5 future 3-5Y revenue-CAGR hypotheses:

```text
H0 <0%, H1 0-5%, H2 5-12%, H3 12-25%, H4 25-50%, H5 >50%
```

For TAM-Adj-PEG:

```text
TAM-Adj-PEG = Forward PE / (EPS CAGR x TAM Runway Factor x Quality Factor)
```

For Prism, preserve lens separation:

- Seri: supply chain only.
- 道士: macro and liquidity only.
- Cat: execution and technical risk only.

## Failure Modes

If current data cannot be fetched, say what is missing and proceed only with
clearly marked stale or user-provided data.

If technical data is unavailable, do not invent DMA or ATR. Ask for a chart or
state that the GF-DMA and Cat sections are incomplete.

If a company has no meaningful EPS or forward P/E, mark TAM-Adj-PEG as distorted
and use normalized or scenario valuation.
