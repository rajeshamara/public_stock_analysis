# Data Layout

## Purpose

This repository should evolve through use rather than trying to design the perfect structure on day one.

> Start with a useful structure, collect real forecasts and outcomes, then change the structure when the data shows a better design.

The repository is stock-centered because the same parameter can have very different predictive importance for different companies.

## Tracked Stock Universe

The authoritative list of tracked stocks is:

`stocks/index.json`

Agents and scheduled jobs must read this file rather than discover stocks by scanning directories.

A stock may be tracked for research without being actively traded.

The public edition has no active stock universe. Copy the synthetic configuration from `examples/` to local `stocks/` as described in the README. Local research files under `stocks/` are ignored by Git. Adding a stock requires updating the local index and creating its corresponding folder and `stock.json` metadata file.

## Template-Driven Agent Architecture

Scheduled tasks must not contain duplicated research logic.

The canonical scheduled-agent entry point is:

`template/stock_agent.md`

A scheduled task should supply only:

- template path
- `TICKER`
- `MODE`

Shared methodology stays in version-controlled repository files. Ticker-specific exceptions belong under `stocks/<folder>/` and should remain exceptions rather than the normal design.

## Directory Structure

```text
/
  RESEARCH_CHARTER.md
  PARAMETER_REGISTRY.md
  FORECAST_PROTOCOL.md
  INTRADAY_FORECAST_PROTOCOL.md
  DATA_LAYOUT.md

  skills/
    stock-forecast/
      SKILL.md

  template/
    README.md
    stock_agent.md
    daily_analysis.md

  schemas/
    forecast_record_v1.example.json
    outcome_record_v1.example.json
    intraday_checkpoint_v1.example.json
    daily_rollup_v1.example.json

  stocks/
    index.json

    <STOCK>/
      stock.json

      daily/
        checkpoints/
          YYYY/
            MM/
              YYYY-MM-DD/
                0800.json
                0900.json
                1000.json
                1045.json
                1100.json
                1200.json
                1300.json
                1400.json
                1500.json
                1615.json

        rollups/
          YYYY/
            MM/
              YYYY-MM-DD.json

        analysis/
          YYYY/
            MM/
              YYYY-MM-DD.md

      expiration/
        forecasts/
          YYYY/
            MM/
              <forecast_id>.json
        outcomes/
          YYYY/
            MM/
              <forecast_id>.json

      analysis/
      research/
```

The `stocks/` portion of this layout is local output, not published data. Nested directories appear when records are created. The public `schemas/` directory contains example instances rather than executable JSON Schema validators. Earlier `daily_forecast` and `daily_outcome` examples use a separate `daily/forecasts/` layout and are retained only as legacy format references.

`BRK-B` is used as the filesystem folder name because `/` cannot be part of a directory name. The canonical ticker remains `BRK/B` inside JSON data.

## Intraday Checkpoint Storage

Use:

`stocks/<folder>/daily/checkpoints/YYYY/MM/YYYY-MM-DD/<HHMM>.json`

Each checkpoint is immutable once written.

A pre-close checkpoint forecasts six first-class targets:

- current-day close
- current-day high
- current-day low
- next-day close
- next-day high
- next-day low

The current methodology also records probability ranges, confidence, reasoning, source summaries, parameter snapshots, revisions from prior checkpoints, and timing of expected high/low when supportable.

## Daily Rollup Storage

After the market closes, create:

`stocks/<folder>/daily/rollups/YYYY/MM/YYYY-MM-DD.json`

The rollup consolidates checkpoint references and realized session facts for convenient statistical analysis. Frozen checkpoint JSON remains the authoritative record of what was predicted at each timestamp.

## Daily Analysis Storage

After the market closes, create:

`stocks/<folder>/daily/analysis/YYYY/MM/YYYY-MM-DD.md`

Use:

`template/daily_analysis.md`

This Markdown file is interpretive and can explain forecast evolution, useful/harmful revisions, influential/noisy information, unexpected events, and questions for future testing.

## Expiration Forecast Storage

Use:

`stocks/<folder>/expiration/forecasts/YYYY/MM/<forecast_id>.json`

Each expiration forecast is immutable once created and contains only information available at its forecast timestamp.

## Expiration Outcome Storage

Use:

`stocks/<folder>/expiration/outcomes/YYYY/MM/<forecast_id>.json`

The outcome references the frozen forecast and records the realized result, interval coverage, probability results, material post-forecast events, and error diagnosis.

## Stock-Specific Files

`stocks/<folder>/stock.json` contains relatively static metadata and explicit stock-specific exceptions or human priors.

Stock-specific behavior should not be hard-coded into shared templates or scheduled tasks unless it truly cannot be represented as stock metadata or learned state.

## Why Stock-Level Folders

Every stock is treated as its own research unit.

Cross-stock similarity is discovered later from data rather than assumed in advance.

Keeping each stock's raw history together supports:

- ticker-level calibration
- feature selection
- ablation testing
- regime analysis
- checkpoint comparison
- eventual cross-stock clustering

## JSON / Markdown / CSV Roles

JSON is the primary machine-readable source of truth for:

- tracked-stock configuration
- parameter snapshots
- forecasts
- outcomes
- probability levels
- model metadata
- statistical processing

Markdown is used for:

- methodology
- templates
- reasoning
- research notes
- conclusions
- human-readable analysis

CSV may be generated later as a derived convenience format for flat statistical analysis. It is not the raw source of truth.

## Versioning Philosophy

Every data record contains a schema version.

If a format changes later, historical frozen records are not silently rewritten merely for cosmetic consistency. Migration should occur only when it provides analytical value.

This preserves the evolution of the experiment and prevents hindsight contamination.

## Scheduled-Agent Contract

Preferred scheduled task:

```text
Execute `template/stock_agent.md` from this repository checkout with:
TICKER=<value>
MODE=<value>
```

Load the selected version of the shared research instructions from the checkout on each authorized run. No scheduler or automatic update mechanism is supplied by this repository.
