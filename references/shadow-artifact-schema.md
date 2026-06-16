# Rank History Shadow Artifact Schema

When `stock-shadow-research` runs inside the `trade-bot` repo, persist outputs
under a dated rank-history directory with one subdirectory per ticker:

```text
rank-history/{YYYY-MM-DD}/{TICKER}/report.md
rank-history/{YYYY-MM-DD}/{TICKER}/{TICKER}.json
```

Use the local current date unless the user provides an explicit as-of date.
Uppercase tickers in filenames.

If multiple tickers are supplied, loop over them independently. One ticker
failure must not block the rest of the batch.

Default overwrite behavior: rerunning the same ticker on the same date replaces
that ticker's `report.md` and `{TICKER}.json`. If the user asks to preserve
versions, write timestamped files:

```text
rank-history/{YYYY-MM-DD}/{TICKER}/report-{HHMMSS}.md
rank-history/{YYYY-MM-DD}/{TICKER}/{TICKER}-{HHMMSS}.json
```

## Markdown Report

`report.md` is human-readable and belongs to exactly one ticker directory. Use a
single level-2 heading:

```markdown
## MU — Stock Shadow Research — 2026-06-16

...
```

Default behavior is to overwrite this file on same-day reruns for the same
ticker.

## Per-Ticker JSON

Write one compact JSON file per ticker. This is intended for downstream review
and possible future integration, not direct production trading.

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

If a ticker fails during a multi-ticker batch, still write a JSON file. Include
the standard fields where possible and add an `error` field:

```json
{
  "ticker": "BADTICKER",
  "research_quality": "low",
  "thesis_quality": "low",
  "fundamental_acceleration": "low",
  "industry_chain_importance": "low",
  "technical_entry_quality": "poor",
  "valuation_risk": "high",
  "cyclicality_risk": "high",
  "winner_preservation_flag": false,
  "new_entry_bias": "avoid",
  "existing_position_bias": "trim_or_exit",
  "thesis_break_risk": "high",
  "thesis_break_triggers": [],
  "error": "Ticker could not be resolved or current data could not be fetched."
}
```

## Field Semantics

- `research_quality`: confidence in source coverage and analysis completeness.
- `thesis_quality`: strength and maturity of the investable thesis.
- `fundamental_acceleration`: reported/guided growth speed.
- `industry_chain_importance`: supply-chain bottleneck or profit-pool relevance.
- `technical_entry_quality`: current entry quality from DMA/ATR structure.
- `valuation_risk`: risk that valuation already prices the thesis.
- `cyclicality_risk`: risk from industry cycle reversal or peak-cycle earnings.
- `winner_preservation_flag`: true when existing winners deserve system-rule
  preservation even if new entries should wait.
- `new_entry_bias`: stance for a fresh position.
- `existing_position_bias`: stance for a currently held position.
- `thesis_break_risk`: probability that the core thesis can break within the
  next few quarters.
- `thesis_break_triggers`: concrete observable events that would invalidate or
  materially weaken the thesis.

## Allowed Values

Use these compact values for consistency:

```yaml
research_quality: low | medium | high
thesis_quality: low | medium | high
fundamental_acceleration: low | medium | high | very_high
industry_chain_importance: low | medium | high | very_high
technical_entry_quality: poor | mixed | good | excellent
valuation_risk: low | medium | medium_high | high
cyclicality_risk: low | medium | medium_high | high
winner_preservation_flag: boolean
new_entry_bias: avoid | wait_for_pullback | small_test_only | acceptable | strong
existing_position_bias: trim_or_exit | hold_with_tight_risk | hold_with_system_rules | add_only_on_confirmation
thesis_break_risk: low | medium | medium_high | high
```

Do not let this JSON directly change production pool selection or order sizing
unless the user explicitly asks for active integration and the repo has a design
or ADR covering that promotion.
