# Forecast Protocol

Version: `0.1`

This document defines how an expiration-price research forecast should be produced, frozen, evaluated, and learned from. It is a research specification; the public repository supplies no forecasting runtime or historical performance results. All numerical examples below are illustrative.

The goal is to create a repeatable walk-forward experiment rather than a sequence of informal market opinions.

---

## Primary Research Question

At a specific forecast timestamp `t`, using only information available at that moment, estimate the probability distribution of the underlying stock price at expiration `T`.

Conceptually:

```text
P(S_T | X_t)
```

where:

- `S_T` = underlying stock price at expiration close
- `X_t` = all information available at forecast time

For a short put strike `K`, derive:

```text
P(S_T >= K)
```

The model's job is to estimate the underlying's expiration distribution.

The trader's job is to decide whether a particular option premium is attractive enough for the associated probability.

---

# 1. Forecast Target

## 1.1 Primary Target

The initial primary target is:

> **Official regular-session closing price of the underlying on the target expiration date.**

For normal US equity sessions this is approximately the 4:00 PM ET close.

This gives the research project one consistent label.

## 1.2 OTM Event Definition

For a short put with strike `K`:

```text
OTM at expiration close = S_T > K
```

If `S_T == K`, record the equality explicitly rather than silently treating it as clearly OTM.

## 1.3 Assignment Versus Forecast Target

The primary research target is the regular-session expiration close, not assignment outcome.

Assignment can occasionally differ because of post-close price changes or exercise decisions. If that becomes important, assignment will be tracked as a separate secondary outcome rather than changing the primary label.

---

# 2. Configured Tickers

Resolve the research universe from the local `stocks/index.json` file. The public repository supplies an inactive, fictional configuration under `examples/` and no default live watchlist.

Each configured ticker uses the same master parameter registry but may develop different feature importance and calibration. Activate a symbol only after the required local metadata and authorized data sources are configured.

---

# 3. Forecast Horizons

Initial emphasis:

- 0-DTE forecasts
- 1-DTE forecasts

The protocol may later expand to longer horizons, but different horizons must never be mixed without a horizon identifier.

Examples:

```text
Tuesday -> Wednesday expiration = 1-DTE style forecast
Wednesday -> Wednesday expiration = 0-DTE forecast
Thursday -> Friday expiration = 1-DTE style forecast
Friday -> Friday expiration = 0-DTE forecast
```

Monday-expiration opportunities may also be analyzed when relevant. The exact number of trading sessions and hours to expiration must always be recorded.

---

# 4. Forecast Timing

One hypothesis is that ordinary-session direction may become easier to forecast around 10:30-11:00 AM ET.

This is a hypothesis to test, not a rule assumed to be true.

## 4.1 Initial Primary Forecast Window

For the first phase of the experiment, prefer a primary forecast around:

```text
10:35 AM - 10:55 AM ET
```

when practical.

The exact timestamp must always be stored.

## 4.2 Forecasts at Other Times

Forecasts may be generated outside this window when useful, but they must be treated as separate observations because available information and remaining time differ.

Examples:

- 9:45 AM forecast
- 10:45 AM forecast
- 12:30 PM forecast
- 2:30 PM forecast

Never compare these as though they are identical forecast conditions.

## 4.3 Multiple Forecasts on the Same Ticker/Expiration

A forecast may be updated during the day, but **never overwrite the earlier forecast**.

Each revision receives a new Forecast ID.

This lets the project answer questions such as:

- Does accuracy improve after 10:30 AM?
- Does a noon update add meaningful information?
- Do late-day forecasts become more accurate only because less time remains?

---

# 5. Forecast ID

Every prediction should have a unique immutable identifier.

Recommended format:

```text
YYYYMMDD-TICKER-EXPIRY-HHMM-HORIZON
```

Example:

```text
20200102-EXAMPLE-20200102-1045-0DTE
20200102-EXAMPLE-20200103-1045-1DTE
```

If two forecasts are generated at the same minute, append a revision suffix.

Example:

```text
20200102-EXAMPLE-20200102-1045-0DTE-R2
```

---

# 6. Information Cutoff

This is a hard research rule.

> **Only information available at or before the forecast timestamp may influence the forecast.**

If an article was published after the forecast, it belongs in postmortem analysis, not in the original forecast rationale.

If a source is discovered later but was already public before the cutoff, it may be noted during audit, but the original frozen prediction must not be edited.

---

# 7. Required Pre-Forecast Data Collection

The forecasting process should attempt to gather the applicable parameters from `PARAMETER_REGISTRY.md`.

The analysis should cover at least these classes when available:

1. target-stock price state,
2. intraday momentum and range,
3. volume,
4. implied volatility / expected move,
5. broad-market direction,
6. sector / peer behavior,
7. VIX / VXN / short-term volatility context,
8. Treasury-yield movement,
9. scheduled macro/Fed events before expiration,
10. company-specific news,
11. analyst actions,
12. regulatory/legal developments,
13. earnings proximity,
14. unusual external/geopolitical developments,
15. historical/ticker-specific context.

Unavailable parameters are marked `NA`; they are not guessed.

---

# 8. News Research Protocol

News interpretation is one of the central reasons for using an LLM.

## 8.1 Search Broadly

Search across multiple independent source classes when useful:

- primary company releases / filings,
- major financial news services,
- credible newspapers/business publications,
- analyst-action reporting,
- regulator/government sources,
- macroeconomic calendars and official releases,
- relevant sector/competitor news.

## 8.2 De-duplicate Events

If 20 sites repeat one original event, treat it as one event.

Record:

- original event,
- publication time,
- source quality,
- novelty,
- materiality,
- direction,
- whether price action confirms it.

## 8.3 Separate Fact from Interpretation

For each meaningful event, distinguish:

```text
FACT: what actually happened
INTERPRETATION: likely market implication
OBSERVED RESPONSE: what the stock/market did after the event, up to forecast time
```

This helps prevent a narrative from being mistaken for evidence.

---

# 9. Regime Classification

Every forecast should assign an initial regime label.

Version 0.1 categories:

- `NORMAL_QUIET`
- `NORMAL_TRENDING`
- `HIGH_VOLATILITY`
- `COMPANY_NEWS`
- `MACRO_EVENT`
- `EARNINGS_RELATED`
- `MARKET_SHOCK`
- `MIXED_OR_UNCLEAR`

Multiple tags may be stored if needed, but one primary regime should be selected.

These labels are intentionally simple. Later clustering may discover better regimes from the data.

---

# 10. Required Forecast Output

Every frozen forecast should contain the following.

## 10.1 Identification

```text
Forecast ID:
Ticker:
Forecast timestamp ET:
Expiration date:
Horizon:
Model/version:
Parameter-registry version:
Forecast-protocol version:
```

## 10.2 Current State

```text
Spot price:
Previous close:
Today's open:
Intraday high so far:
Intraday low so far:
Primary regime:
```

## 10.3 Distribution Forecast

At minimum:

```text
Central estimate:
50% probability interval:
80% probability interval:
95% probability interval:
```

These intervals should be nested.

Example:

```text
Central estimate: $331.70
50% interval: $330.80 - $332.60
80% interval: $329.20 - $334.00
95% interval: $327.10 - $336.40
```

## 10.4 Practical Price-Level Probabilities

Provide probabilities at multiple nearby levels so the trader can compare strikes.

Example:

```text
P(close >= $333): 35%
P(close >= $332): 49%
P(close >= $331): 64%
P(close >= $330): 82%
P(close >= $329): 90%
P(close >= $328): 95%
P(close >= $327): 97%
```

The price ladder should be relevant to the ticker and current price, not mechanically identical across stocks.

## 10.5 Direction Probabilities

Record:

```text
P(close > current spot):
P(close approximately unchanged):
P(close < current spot):
```

The exact definition of "approximately unchanged" should eventually be standardized; for version 0.1 use a small ticker-appropriate band and state it explicitly.

## 10.6 Drivers

List the strongest current influences, not every observed parameter.

```text
Primary bullish drivers:
Primary bearish drivers:
Neutral/noisy factors:
Scheduled risk before expiration:
Unexpected-news risk: LOW / NORMAL / ELEVATED
```

## 10.7 Confidence

Use:

- `LOW`
- `MODERATE`
- `HIGH`

Confidence is **not** probability of being correct.

It reflects how stable and well-supported the estimated distribution appears given the available evidence.

---

# 11. Human Forecast

When practical, record the trader's independent expectation **before seeing the LLM forecast**.

Suggested minimal form:

```text
Human central estimate:
Human likely range:
Human directional view:
Human confidence:
```

This enables three long-term comparisons:

1. human alone,
2. LLM alone,
3. combined decision process.

The purpose is not competition; it is to learn where each source of judgment adds value.

---

# 12. Freeze Rule

Once a live forecast is issued:

> **The original forecast values must never be edited to make them look better after the fact.**

Corrections to formatting or obvious transcription errors may be appended, but the original prediction remains visible.

If new information requires a revised outlook, create a new Forecast ID.

This is essential for honest calibration.

---

# 13. Outcome Capture

After expiration, record at minimum:

```text
Actual official close:
Return from forecast spot to close:
Absolute central-estimate error:
Percentage central-estimate error:
Was actual close inside 50% interval? yes/no
Was actual close inside 80% interval? yes/no
Was actual close inside 95% interval? yes/no
```

Also record post-forecast path statistics when available:

```text
Highest price after forecast:
Lowest price after forecast:
Maximum favorable excursion:
Maximum adverse excursion:
```

For every published price-level probability, record whether the event occurred.

Example:

```text
Forecast: P(close >= $330) = 84%
Outcome: YES
```

That creates the data required for probability calibration.

---

# 14. Postmortem Error Classification

Every meaningful miss should be analyzed, but not every miss should cause a methodology change.

Initial error categories:

- `DIRECTION_ERROR`
- `VOLATILITY_UNDERESTIMATED`
- `VOLATILITY_OVERESTIMATED`
- `NEWS_WEIGHTING_ERROR`
- `MARKET_FACTOR_WEIGHTING_ERROR`
- `RATE_FACTOR_WEIGHTING_ERROR`
- `SECTOR_FACTOR_WEIGHTING_ERROR`
- `INTRADAY_SIGNAL_ERROR`
- `EVENT_RISK_ERROR`
- `UNEXPECTED_POST_FORECAST_NEWS`
- `NORMAL_RANDOM_VARIATION`
- `DATA_QUALITY_ERROR`
- `MODEL_OVERCONFIDENCE`
- `MODEL_UNDERCONFIDENCE`
- `UNKNOWN`

A postmortem should answer:

1. What happened?
2. Which forecast component was wrong?
3. Which parameters appear most relevant?
4. Was the event knowable at forecast time?
5. Should any parameter weight/heuristic be reconsidered?
6. Is this one observation or evidence of a repeated pattern?

---

# 15. Unexpected Shock Rule

An event occurring after the forecast can make a prediction wrong without proving the original reasoning was poor.

Examples include:

- surprise company announcement,
- unexpected regulatory action,
- sudden geopolitical escalation,
- unscheduled Fed communication,
- market-wide technical disruption.

These forecasts still count as forecast misses.

They are **not removed from accuracy statistics**.

However, tag them separately so the project can later compare:

```text
all forecasts
vs.
forecasts without material post-forecast shocks
```

---

# 16. Calibration Tests

Calibration is a central objective.

If the system issues many events at approximately 80% probability, approximately 80% should occur over a sufficiently large sample.

Track probability buckets such as:

- 50-59%
- 60-69%
- 70-79%
- 80-89%
- 90-94%
- 95-100%

For each bucket, compare:

```text
Average predicted probability
Actual observed frequency
Number of observations
```

Useful metrics may include:

- Brier score,
- calibration error,
- reliability curves,
- log loss where appropriate,
- interval coverage.

---

# 17. Interval Coverage Tests

Over a large sample:

- approximately 50% of actual closes should fall inside the stated 50% intervals,
- approximately 80% should fall inside 80% intervals,
- approximately 95% should fall inside 95% intervals.

If 95% intervals contain 100% of outcomes, the system may be too conservative.

If they contain only 75%, the system is materially overconfident.

---

# 18. Feature-Learning Process

Feature importance should emerge gradually from observed performance.

Do not change the process aggressively after one unusual day.

As observations accumulate, explore:

- correlation,
- Spearman correlation,
- mutual information,
- logistic/linear models,
- LASSO / Elastic Net,
- tree-based models,
- permutation importance,
- SHAP-style attribution,
- ablation testing,
- clustering / regime discovery.

Example ablation question:

```text
Does removing QQQ context materially reduce AAPL forecast accuracy?
```

If not, its apparent importance may be narrative rather than predictive.

---

# 19. Walk-Forward Rule

The experiment is strictly chronological.

Correct process:

```text
Use past data -> forecast future -> observe result -> learn -> next forecast
```

Do not use future observations to tune a model and then report performance on an earlier period as though it were live.

When formal statistical models are introduced, use walk-forward validation or appropriately nested time-series validation.

Random train/test splits should not be the primary validation method for this time-dependent problem.

---

# 20. Model Version Tracking

The underlying LLM may change over the years.

Every forecast must record the model used.

Examples:

```text
<provider/model-version>
future model version
```

This allows later analysis such as:

```text
Did forecasting improve after a model upgrade?
Did calibration change?
Did news interpretation improve while price-range estimation remain unchanged?
```

A model upgrade should not erase earlier history; it creates a new era in the same experiment.

---

# 21. Methodology Version Tracking

Model version and research-method version are different.

Record both.

For example:

```text
Model: <provider/model-version>
Forecast protocol: 0.1
Parameter registry: 0.1
```

If the forecasting process changes materially, increment the protocol version.

This prevents later improvement from being mistakenly attributed solely to a new LLM.

---

# 22. Performance Metrics

The research project should evaluate forecasting quality separately from trading results.

## 22.1 Forecast Metrics

Examples:

- mean absolute error of expiration close,
- median absolute error,
- directional accuracy,
- probability calibration,
- interval coverage,
- Brier score,
- error by ticker,
- error by DTE,
- error by weekday,
- error by forecast time,
- error by regime,
- error by model version.

## 22.2 Trading Metrics

Separately track:

- weekly realized option income,
- number of trades,
- number of contracts,
- average premium per contract,
- number of ITM expirations,
- number of rolls,
- duration of rolled positions,
- realized losses,
- capital/risk tied up in long-duration rolls.

Do not allow good trading results to prove a bad forecast was accurate, or a good forecast to hide poor trade construction.

---

# 23. Research and Trading Outcomes

Forecast research should be evaluated independently of personal income goals, account holdings, or realized trading results. None of those private records is required by this public framework.

A forecast must not be changed to justify a trade or a target return. Any separate analysis of trading results belongs outside the public research templates and requires its own assumptions and evaluation.

---

# 24. User-Defined Priors

Users may document independent stock-specific hypotheses in their local metadata. Record when and why a prior was formed, keep it separate from model output, and evaluate whether future observations support it.

The public edition assumes no ranking of individual stocks, preferred strike distance, position size, or rolling strategy. These are not prerequisites for forecasting the underlying price distribution.

---

# 25. Initial Intraday Hypotheses

Test explicitly:

1. By roughly 10:30-11:00 AM ET, daily direction is materially easier to forecast than near the open.
2. Session highs frequently occur around the 10:30-11:00 AM period for the selected tickers under ordinary conditions.
3. Friday is less volatile than other trading days absent major news.
4. The importance of these patterns varies by ticker.
5. Company news and macro events can dominate or invalidate ordinary intraday behavior.

Do not convert these hypotheses into facts until supported by sufficient observations.

---

# 26. Forecast Template

Use this as the initial human-readable format.

```markdown
# Forecast: <Forecast ID>

## Identification
- Ticker:
- Forecast timestamp ET:
- Expiration:
- Horizon:
- Model:
- Protocol version:
- Parameter registry version:

## Market State
- Spot:
- Previous close:
- Open:
- Intraday high:
- Intraday low:
- Primary regime:

## Human Prior
- Central estimate:
- Likely range:
- Directional view:
- Confidence:

## LLM Forecast
- Central estimate:
- 50% interval:
- 80% interval:
- 95% interval:

### Price-Level Probabilities
| Price level | P(close >= level) |
|---:|---:|
| | |

### Direction
- P(close above current spot):
- P(close roughly unchanged):
- P(close below current spot):

## Drivers
### Bullish
-

### Bearish
-

### Neutral / Noise
-

### Scheduled Risks Before Expiry
-

### Unexpected-News Risk
-

## Key Parameters / Evidence
-

## Outcome — fill after expiration
- Actual close:
- Absolute central-estimate error:
- Inside 50% interval:
- Inside 80% interval:
- Inside 95% interval:
- Post-forecast high:
- Post-forecast low:

## Postmortem
- Error classification:
- What mattered most:
- What was missed:
- Was the miss knowable at forecast time?:
- Candidate learning:
```

---

# 27. Research Discipline

The experiment should remain comfortable with all possible conclusions.

Possible long-term outcomes include:

- strong useful forecasting improvement,
- modest improvement only for certain tickers,
- improvement only in quiet/no-news regimes,
- human judgment outperforming the LLM on some tickers,
- LLM synthesis outperforming human intuition on others,
- implied options data containing nearly all useful short-horizon information,
- or no durable predictive advantage at all.

The experiment succeeds if it discovers the truth more reliably than intuition alone.

---

# 28. Next Implementation Documents

Possible future artifacts (not implemented in this repository):

- `DATA_SCHEMA.md` — exact machine-readable forecast/outcome structure
- `NEWS_SCORING.md` — rules for P81-P90 and event de-duplication
- `PEER_BASKETS.md` — fixed P54 peer definitions by ticker
- `FORECASTS/` — immutable individual forecast records
- `POSTMORTEMS/` — detailed error analyses when useful
- `ANALYSIS/` — periodic calibration and feature-importance studies

These should be added only as the process reaches the point where they are useful rather than creating documentation with no operational purpose.
