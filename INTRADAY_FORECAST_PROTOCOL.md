# Intraday Forecast Protocol

## Purpose

This protocol defines the repeated intraday forecasting loop for the stock research project. It specifies a workflow; a data source, runtime, numerical evaluator, and scheduler are not implemented here. Path examples use the fictional symbol `EXAMPLE`.

The objective is to observe how forecast quality changes as new information arrives during the trading day. Each checkpoint is a fresh, frozen prediction made only from information available at that timestamp.

The system forecasts both:

1. the current trading day's final close, final high, and final low; and
2. the next trading day's close, high, and low.

**Close, high, and low are three equal first-class forecast targets.** High and low must not be treated as optional supporting fields simply because closing price is currently the most obvious options-related target. Their future research value is unknown, which is exactly why they are captured from the beginning.

After the current session has closed, the current-day targets are no longer forecast. The closing checkpoint evaluates the completed day and forecasts only the next trading day's close, high, and low.

## Why Repeated Intraday Forecasts Matter

A single daily prediction does not tell us when information becomes useful.

Repeated checkpoints let the project measure:

- how quickly uncertainty falls during the day,
- whether the first 30-90 minutes materially improve accuracy,
- whether specific hours contain more predictive information than others,
- when a forecast becomes overconfident,
- how news changes the forecast path,
- how much the forecast changes because of price action versus external information,
- whether high/low forecasts contain information that later improves close or expiration forecasts,
- whether the timing and magnitude of the eventual high/low reveal ticker-specific regimes,
- and whether a stable subset of parameters consistently drives revisions.

The forecast path itself becomes research data.

## Initial V1 Checkpoints

Start with the following Eastern Time checkpoints on normal U.S. trading days:

- 08:00 - premarket baseline
- 09:00 - later premarket update
- 10:00 - 30 minutes after market open
- 10:45 - special checkpoint retained to test a hypothesis about forecast quality around 10:30-11:00 AM
- 11:00 - hourly checkpoint
- 12:00 - midday
- 13:00 - early afternoon
- 14:00 - afternoon
- 15:00 - final full-hour checkpoint
- 16:15 - post-close evaluation and next-day forecast

This cadence is deliberately provisional. It may be changed later based on evidence.

## First-Class Target Rule

At every checkpoint, treat each target below independently:

### Current trading day

- final close
- final high
- final low

### Next trading day

- close
- high
- low

For **each** target, record:

- a central estimate,
- a 50% interval,
- an 80% interval,
- a 95% interval,
- confidence / uncertainty,
- and the reasons that most influenced the estimate.

Where useful, level probabilities may also be recorded for high and low, for example:

- `P(final high >= X)`
- `P(final low <= Y)`

Do not assume in advance that high/low predictions are less useful than close predictions. Their value will be determined later from accumulated evidence.

## Targets Before the Close

At every checkpoint before the market closes, produce forecasts for both horizons below.

### Horizon A - Current Trading Day

Predict:

- final closing price,
- final intraday high,
- final intraday low,
- 50%, 80%, and 95% intervals for final close,
- 50%, 80%, and 95% intervals for final high,
- 50%, 80%, and 95% intervals for final low,
- probability of closing above the previous close,
- probability of closing above the current price at forecast time,
- directional probabilities,
- expected remaining-day volatility,
- and confidence / uncertainty.

If the day's realized high or low has already occurred, the predicted final high/low and all corresponding intervals must respect the information already observed. For example, the predicted final high and the lower bound of a final-high interval cannot be below the high already reached. Likewise, the predicted final low and the upper logic of the low forecast must respect a low already reached.

### Horizon B - Next Trading Day

Predict:

- next trading day's closing price,
- next trading day's intraday high,
- next trading day's intraday low,
- 50%, 80%, and 95% intervals for the next close,
- 50%, 80%, and 95% intervals for the next high,
- 50%, 80%, and 95% intervals for the next low,
- probability next close is above the current day's eventual close when that reference can be estimated,
- directional probabilities,
- and confidence / uncertainty.

At pre-close checkpoints the next-day forecast is necessarily conditional on an unknown current-day close. Record the exact reference assumptions used.

## Post-Close Checkpoint

At approximately 16:15 ET:

1. record the completed session's actual open, high, low, close, and volume when available;
2. evaluate every frozen current-day close/high/low forecast checkpoint;
3. measure close error, high error, and low error by checkpoint;
4. evaluate interval coverage separately for close, high, and low;
5. identify major forecast revisions and why they occurred;
6. identify post-forecast information that materially changed the market;
7. forecast only the next trading day's close, high, low, ranges, and directional probabilities.

The post-close forecast becomes the first prediction in the next day's forecast path.

## Required Rationale at Every Checkpoint

Every checkpoint must explain why the forecast has its current values.

Capture at minimum:

- the most influential bullish inputs,
- the most influential bearish inputs,
- neutral or conflicting inputs,
- important parameter changes since the prior checkpoint,
- new company-specific news,
- new sector or peer developments,
- broad-market movement,
- volatility / options-implied changes when available,
- rates / macro changes when relevant,
- scheduled events before the target horizon,
- major uncertainty,
- reasons behind the close estimate,
- reasons behind the high estimate,
- reasons behind the low estimate,
- and the reason each forecast changed or stayed stable versus the prior checkpoint.

Do not merely record numbers. The reasoning history is part of the research dataset.

## Incremental Research Rule

The 08:00 checkpoint should perform the broadest baseline information sweep.

Later checkpoints should first read the prior checkpoint and focus research effort on what changed since then:

- new price action,
- new market movement,
- new news,
- new analyst actions,
- new macro information,
- changes in volatility,
- changes in rates,
- changes in sector / peers,
- and newly relevant scheduled-event risk.

Do not reread and recount the same unchanged story as a new independent signal.

## Data Storage Model

The source-of-truth forecast snapshots remain immutable checkpoint JSON files.

Use:

`stocks/<folder>/daily/checkpoints/YYYY/MM/YYYY-MM-DD/<HHMM>.json`

Examples:

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/0800.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1045.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1500.json`

`stocks/EXAMPLE/daily/checkpoints/2020/01/2020-01-02/1615.json`

Each checkpoint is written once and never altered later.

## Daily Machine-Readable Rollup

After the market closes, create a consolidated JSON file:

`stocks/<folder>/daily/rollups/YYYY/MM/YYYY-MM-DD.json`

The rollup should reference or summarize all frozen checkpoint records and contain the actual session outcome.

The rollup must retain close/high/low predictions and errors independently so later analysis can determine whether one target is more predictable or more useful than another.

The rollup is for convenient statistical processing. The immutable checkpoint files remain the authoritative record of what was predicted at each timestamp.

## Daily Human-Readable Analysis

After the market closes, create or update:

`stocks/<folder>/daily/analysis/YYYY/MM/YYYY-MM-DD.md`

This Markdown file should explain the day's forecast evolution in human-readable form.

It should include:

- what the model believed at 08:00,
- how the close/high/low forecasts changed through the day,
- which inputs caused major revisions,
- actual open/high/low/close,
- which checkpoints were most accurate for close,
- which checkpoints were most accurate for high,
- which checkpoints were most accurate for low,
- which checkpoints were least accurate,
- why the misses likely occurred,
- unexpected events,
- parameters that appeared influential,
- parameters that appeared noisy,
- and questions to test in future sessions.

The analysis file may be refined after close because it is interpretation, not a frozen prediction. The underlying checkpoint JSON files must remain unchanged.

## CSV Policy

CSV is not the primary source of truth because nested probability ranges, parameter snapshots, source metadata, and reasoning do not fit cleanly into one flat row.

A CSV dataset may be generated later as a derived analytical artifact containing selected fields such as:

- ticker,
- date,
- checkpoint,
- spot price,
- predicted current close,
- predicted current high,
- predicted current low,
- predicted next close,
- predicted next high,
- predicted next low,
- actual close,
- actual high,
- actual low,
- close error,
- high error,
- low error,
- model version,
- regime,
- and selected parameter values.

JSON remains the raw machine-readable record; Markdown remains the narrative research record; CSV may become a derived statistical convenience format.

## Forecast Revision Analysis

For every checkpoint after the first, record revisions separately for all six targets:

- prior and new predicted current-day close,
- prior and new predicted current-day high,
- prior and new predicted current-day low,
- prior and new predicted next-day close,
- prior and new predicted next-day high,
- prior and new predicted next-day low,
- revision amounts for each,
- and the top reasons for each revision.

This allows later analysis of whether specific information types improve forecasts or merely cause unnecessary forecast movement.

## Long-Term Questions Enabled by This Protocol

Over time the project should be able to answer questions such as:

- Is 08:00 forecasting materially useful?
- How much does accuracy improve after the open?
- Is 10:45 consistently more informative than 10:00 or 11:00?
- At what time does current-day closing-price error become acceptably small?
- At what time do the day's eventual high and low become meaningfully predictable?
- Does predicting the high/low improve expiration-level probability estimates?
- Does next-day forecasting improve or degrade as the current session progresses?
- Which parameters most often trigger useful forecast revisions?
- Which parameters cause revisions that later prove wrong?
- Which stocks become predictable earlier in the day?
- Which stocks remain noisy even near the close?
- Does a model's confidence calibrate correctly by time of day and by target type?

These questions should be answered from accumulated data rather than intuition alone.
