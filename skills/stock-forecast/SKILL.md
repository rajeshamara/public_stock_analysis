---
name: stock-forecast
description: Prepare and evaluate stock-forecast research records using this repository's protocols, templates, and user-configured authorized data sources. Use for explicit intraday or expiration-price research and post-close analysis; this skill does not execute trades or install schedules.
---

# Stock Forecast Research Skill

## Purpose

This file is the shared operating procedure for scheduled and interactive stock-forecast research in this repository.

The process is intentionally versionable and iterative. Start with a useful system, collect real observations, measure results, and change the methodology only when evidence or operational experience justifies it.

## Scope and Prerequisites

Resolve repository file paths relative to the checkout root. Follow the active system, developer, and user instructions; repository methodology does not override permissions or user constraints.

Use only user-authorized data sources and an explicitly selected runtime. No provider integration, forecasting engine, scheduler, or brokerage connection is included here. Do not place trades, publish local records, or create schedules as part of this skill. Keep credentials outside committed files and research output in the ignored local `stocks/` directory.

Stop for inactive or missing configuration. If required inputs are unavailable, preserve that limitation and produce no unsupported normal forecast. The public examples use a fictional symbol and null measurements and must never be presented as observations or predictions.

## Template-Driven Execution

Scheduled stock agents must not duplicate the research logic contained in this repository.

The canonical scheduled-agent wrapper is:

`template/stock_agent.md`

A scheduled task should provide only the template path and its input values, normally:

- `TICKER`
- `MODE`

The shared methodology belongs in this skill and the referenced repository protocols. Ticker-specific behavior belongs under the stock's own folder and should be the exception rather than the norm.

Do not copy this skill, forecasting rules, parameter definitions, schema definitions, storage rules, or research methodology into individual scheduled-task prompts.

## Single Source of Truth for Tracked Stocks

Always begin by reading:

`stocks/index.json`

Do **not** discover the research universe by scanning stock folders.

Only symbols with `research_status = "active"` should be processed by default.

The `folder` field in `stocks/index.json` determines where stock-specific data is written.

## Required Context

Before generating forecasts or evaluating outcomes, read:

1. `RESEARCH_CHARTER.md`
2. `PARAMETER_REGISTRY.md`
3. `FORECAST_PROTOCOL.md`
4. `INTRADAY_FORECAST_PROTOCOL.md`
5. `DATA_LAYOUT.md`
6. `stocks/index.json`

Use the current schema examples under `schemas/` and reusable templates under `template/`.

## Research Problems

The project tracks three related forecasting problems.

### 1. Intraday Forecast Path

At repeated checkpoints during the day, forecast both:

- the current trading day's final close, final high, and final low; and
- the next trading day's close, high, and low.

**Close, high, and low are equal first-class targets.** Do not downgrade high or low to optional supporting fields just because closing price is currently the most obvious options-related target. Their future analytical value is unknown and must be tested from data.

At the post-close checkpoint, the current session is already known, so evaluate it and forecast only the next trading day.

The initial V1 checkpoints are:

- `0800`
- `0900`
- `1000`
- `1045` - special checkpoint retained to test the 10:30-11:00 AM hypothesis
- `1100`
- `1200`
- `1300`
- `1400`
- `1500`
- `1615` - post-close evaluation and next-day forecast

These times are hypotheses, not permanent rules.

### 2. Daily Close / High / Low Forecasting

For the current day, track separately for **close**, **high**, and **low**:

- central estimate
- 50%, 80%, and 95% intervals
- confidence / uncertainty
- target-specific rationale

Also track:

- directional probabilities
- probability close is above previous close
- probability close is above forecast-time spot
- expected remaining-day volatility

For the next trading day, track separately for **close**, **high**, and **low**:

- central estimate
- 50%, 80%, and 95% intervals
- confidence / uncertainty
- target-specific rationale

Also track:

- directional probabilities
- reference assumptions when the current day's close is not yet known

Where useful, capture level probabilities such as `P(high >= X)` and `P(low <= Y)`.

When supportable, also forecast the likely time window of the day's high and low. Timing may later prove useful for options-entry analysis and should be preserved rather than discarded prematurely.

### 3. Expiration Forecasting

Forecast the stock price at the nearest relevant option expiration.

Resolve the target expiration from current authorized reference data for the configured symbol. Do not infer an available expiration from the day of the week alone.

The output should emphasize the underlying's expiration-price distribution, not an option trade recommendation.

At minimum record:

- central expiration-price estimate
- 50%, 80%, and 95% intervals
- probabilities of closing at or above relevant candidate price levels
- directional probabilities
- major drivers
- uncertainty and event risk
- full forecast-time parameter snapshot

The trader decides whether any strike or premium is attractive.

## Stock Isolation

A stock-specific agent operates only on its resolved ticker.

It may read market-wide, macro, peer, sector, cross-asset, volatility, news, and other stock data when those are inputs to defined parameters. It must not create or update another stock's forecast/history files.

Cross-stock relationships are discovered later through separate analysis and are not assumed by the individual stock agent.

## Intraday Checkpoint Storage

Each checkpoint is immutable once written.

Use:

`stocks/<folder>/daily/checkpoints/YYYY/MM/YYYY-MM-DD/<HHMM>.json`

Examples:

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/0800.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1045.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1500.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1615.json`

Use `schemas/intraday_checkpoint_v1.example.json` as the current starting shape.

Never overwrite an earlier checkpoint with a later opinion.

## Daily Rollup Storage

After the market closes, create:

`stocks/<folder>/daily/rollups/YYYY/MM/YYYY-MM-DD.json`

Use `schemas/daily_rollup_v1.example.json` as the current starting shape.

The rollup must retain and evaluate close, high, and low predictions independently.

This file exists for convenient statistical processing. Frozen checkpoint files remain the authoritative prediction records.

## Daily Analysis Storage

After the market closes, create:

`stocks/<folder>/daily/analysis/YYYY/MM/YYYY-MM-DD.md`

Use `template/daily_analysis.md` as a starting format.

The Markdown file should explain how close/high/low forecasts evolved, why revisions occurred, which information mattered, what appeared noisy, and what should be tested later.

Unlike frozen checkpoint JSON, the analysis file is interpretive and may be refined after the close.

## Expiration Forecast Storage

Use:

`stocks/<folder>/expiration/forecasts/YYYY/MM/<forecast_id>.json`

## Expiration Outcome Storage

Use:

`stocks/<folder>/expiration/outcomes/YYYY/MM/<forecast_id>.json`

## Broader Postmortem Storage

If a forecast or event deserves deeper analysis beyond the daily journal, use:

`stocks/<folder>/analysis/YYYY/MM/<forecast_id-or-date>.md`

Do not create verbose special postmortems for every normal observation.

## Market-Closed Rule

If the U.S. equity market is closed for a full-day holiday, do not create normal market-session forecasts or outcomes.

If useful, record only an operational skipped-run marker.

For shortened sessions, use the actual market close and mark the session as shortened.

## Data Collection Rules

1. Use only information available at the forecast timestamp.
2. Search broadly enough to cover company-specific, market, macro, options-implied, sector, peer, and event information that can reasonably influence the horizon.
3. Multiple articles repeating the same underlying story count as one independent news event, not multiple signals.
4. Preserve missing data as missing. Never fabricate a parameter value.
5. Record sources or source summaries sufficiently to reconstruct why a forecast was made.
6. Distinguish scheduled known risks from unexpected post-forecast shocks.
7. Keep raw frozen predictions separate from later interpretation.
8. Record the actual model/version used whenever known.
9. Preserve close, high, and low forecasts even if one later appears less useful; usefulness is an empirical question.

## Incremental Research Rule

The 08:00 checkpoint should perform the broadest baseline information sweep.

Later checkpoints should first read the prior checkpoint and focus on what changed since then:

- price action
- market direction
- company news
- sector / peer movement
- macro / rates
- volatility / options-implied conditions
- scheduled-event risk

Do not repeatedly treat unchanged information as a new signal.

## Revision Tracking

For every checkpoint after the first, explicitly record revisions for all six forecast targets:

- current-day close
- current-day high
- current-day low
- next-day close
- next-day high
- next-day low

For each target record:

- prior estimate
- new estimate
- revision amount
- reasons for the revision or reasons for leaving it unchanged

A forecast that stays unchanged should also explain why new information was insufficient to justify a revision.

## Learning Rules

Do not alter historical checkpoint forecasts to improve apparent accuracy.

After outcomes are known:

- measure close forecast error
- measure high forecast error
- measure low forecast error
- test 50/80/95 interval coverage separately for close/high/low
- test level-probability calibration where recorded
- compare checkpoint accuracy by time of day
- classify major error causes
- note unexpected information that appeared after a forecast
- distinguish useful revisions from harmful revisions
- periodically evaluate which parameters add predictive value by ticker, target type, time of day, and regime

Do not assume a parameter matters merely because it sounds plausible.

## Human Prior

When the trader provides an independent view before seeing the model forecast, preserve it separately from the model forecast.

This enables later comparison of:

- human-only judgment
- model-only judgment
- combined judgment

## JSON / Markdown / CSV Roles

Use JSON as the primary machine-readable source of truth.

Use Markdown for reasoning, interpretation, research notes, and postmortems.

CSV may be generated later as a derived analytical dataset, but it should not replace richer JSON records.

## Operational Principle

The repository stores versioned research context and methodology for future runs.

Future runs should read the current methodology and relevant accumulated results before changing procedures. Methodology changes should be documented and versioned instead of silently changing the historical meaning of fields.

Individual scheduled tasks are execution wrappers only. The repository defines the research workflow within the active instruction hierarchy and authorized scope.
