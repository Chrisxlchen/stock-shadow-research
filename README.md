# Stock Shadow Research Skill

`stock-shadow-research` is a Codex skill that produces repeatable sidecar stock
research reports from one or more tickers. It orchestrates Serenity-style alpha,
GF-DMA trend health, TAM-Adj-PEG valuation, Bayesian intrinsic-growth valuation,
buy-side memo framing, and Prism three-lens synthesis.

It is designed for research review only. It does not modify production pools,
ranking code, order logic, databases, or position sizing.

## Prerequisites

Install the supporting skills first.

### Required

Prism:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo destiny520537work-lab/prism-skill \
  --path . \
  --name prism-skill \
  --method git
```

Serenity skills:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo haskaomni/serenity-skill \
  --path skills/bayesian-intrinsic-growth-valuation \
         skills/buy-side-equity-research-memo \
         skills/gf-dma-health-index \
         skills/serenity-alpha \
         skills/tam-adj-peg \
  --method git
```

### Optional

UZI-Skill can be installed separately if you also want its external deep-analysis
workflow:

```bash
git clone https://github.com/wbh604/UZI-Skill.git ~/.codex/UZI-Skill
cd ~/.codex/UZI-Skill
python3 -m pip install -r requirements.txt
python3 -m playwright install chromium
```

Restart Codex after installing skills.

## Install This Skill

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo <owner>/stock-shadow-research \
  --path . \
  --name stock-shadow-research \
  --method git
```

Replace `<owner>` with the GitHub user or organization that owns the repository.

## Usage

Examples:

```text
Run stock-shadow-research for MU.
Run stock-shadow-research for MU ALAB DDOG.
用 stock-shadow-research 分析 MU。
用 stock-shadow-research 分析 MU、ALAB、DDOG。
```

For multiple tickers, the skill loops over each ticker independently. A failure
for one ticker does not stop the rest of the batch.

## Output

When used inside the `trade-bot` repo, the skill writes:

```text
rank-history/<YYYY-MM-DD>/<TICKER>/report.md
rank-history/<YYYY-MM-DD>/<TICKER>/<TICKER>.json
```

Same-day reruns for the same ticker overwrite that ticker's files by default.
If the user asks to preserve versions, timestamped files are written instead.

The JSON output is compact and intended for downstream review:

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

## Repository Contents

```text
stock-shadow-research/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    └── shadow-artifact-schema.md
```

## Boundary

This skill is a shadow research layer. It may suggest future production fields,
but it must not directly change pool membership, ranking, sizing, or orders
without a separate implementation request and design review.
