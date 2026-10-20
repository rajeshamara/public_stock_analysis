# Stock Forecast Research Charter

## Purpose

Create a repeatable experiment for studying short-horizon stock-price forecasts. The central question is whether information available at a particular timestamp can support useful, calibrated estimates of later prices, and whether those estimates improve on simple baselines.

The public project provides the research design, record shapes, and agent instructions. It does not publish personal financial history, a live research universe, historical results, or a validated forecasting model.

## Research Questions

The framework separates three related questions:

1. How do estimates of the current day's close, high, and low change as information arrives?
2. How do estimates of the next trading day's close, high, and low change across the current day?
3. What distribution can be estimated for the underlying stock's closing price at a selected option expiration?

Close, high, and low are distinct targets. Forecasting one well does not establish that the others are predictable. Expiration-price estimates describe the underlying; trade selection and execution are outside this framework.

## Experimental Cycle

Each observation follows a consistent lifecycle:

1. Identify the stock, target, horizon, forecast time, and method version.
2. Collect authorized data available by that time and record its provenance.
3. Preserve missing or unreliable inputs as missing.
4. Freeze the estimate, uncertainty, rationale, and parameter snapshot.
5. Observe the later outcome without changing the original forecast.
6. Measure error and calibration, diagnose the miss, and record what changed.
7. Apply justified methodology changes only to future observations.

Repeated checkpoints preserve a forecast path. Post-close rollups summarize numerical comparisons, while separate Markdown analysis captures interpretation and questions for later testing.

## Evidence Standards

### Information cutoffs

Use only information available at or before the forecast timestamp. Later news belongs in outcome analysis. Finding an earlier-published source after the fact does not justify silently inserting it into a frozen prediction.

### Immutable forecasts

A revision is a new observation with its own timestamp. Historical forecasts must not be changed to improve apparent accuracy. Outcome records reference the original forecast, and interpretive notes remain separate.

### Missing data

Unavailable measurements are not zero. Use JSON `null` for missing measurements, record missing parameter IDs or reasons where supported, and avoid producing unsupported estimates simply to fill a template.

### Provenance

Record the model identifier, method version, source summaries, and observation timestamps sufficiently to understand the basis of a prediction. Preserve original units and distinguish measured values from derived estimates.

### Independent human priors

An optional human forecast can be recorded before the model output is shown. Keep it separate so that later comparisons can distinguish human-only, model-only, and combined judgment. Personal positions or account details are not required.

## Evaluation Plan

The protocols propose the following evaluation dimensions:

- Absolute and percentage error for each target and horizon.
- Coverage of stated 50%, 80%, and 95% intervals.
- Calibration of level and directional probabilities.
- Differences by stock, regime, checkpoint, and method version.
- Whether revisions improve forecasts or add noise.
- Comparisons with simple, explicitly defined baseline forecasts.
- Performance on later observations not used to choose the method.

These are evaluation requirements and research plans. The public repository does not contain computed results or an implemented scoring engine. Small samples, multiple hypotheses, and changes in conditions must be considered when interpreting any future experiment.

Forecast accuracy and trading results are separate questions. A research claim should be supported by forecast evidence rather than inferred from personal profits, losses, or income goals.

## Hypotheses and Parameter Learning

The parameter registry provides a stable vocabulary for possible inputs. Inclusion in the registry is not evidence that a parameter predicts prices.

Ticker-specific patterns, the value of the 10:45 ET checkpoint, and the usefulness of news, options context, or market-wide indicators are hypotheses. They should be tested against observations and baselines rather than hard-coded as established truths.

Feature selection and method tuning should follow chronological, walk-forward evaluation. Information from a later outcome must not influence a forecast that is presented as having been made earlier. Retain negative findings and unsuccessful hypotheses.

## Versioning and Architecture

Record-format versions, research-method versions, and model identifiers serve different purposes. Keep them separate so that changes can be interpreted correctly.

The shared protocols define the research process. The stock-agent template resolves a configured ticker and isolates its records. Stock-specific metadata belongs in that stock's local directory. Reusable methodology stays in the shared files.

Repository instructions operate within the user's authorized scope and the active runtime's instruction hierarchy. They do not grant access to data, authorize trading, create schedules, or override user constraints.

## Public Scope

The public edition contains:

- This charter and the forecast protocols.
- The 100-parameter registry.
- Example record shapes with fictional identifiers and unpopulated measurements.
- Reusable agent and analysis templates.
- Synthetic, inactive configuration examples.

Local stock research, account information, credentials, and generated exports are excluded from the public workflow. A data source must be authorized by the user and its terms must permit the intended use; the repository supplies no market-data license.

## Successful Outcomes

The experiment may find useful improvement for some targets, conditional value in particular settings, a strong baseline that is difficult to beat, or no durable predictive advantage. Any of those can be a useful result when supported by a sound process.

The goal is an auditable research record and a clearer understanding of uncertainty. The framework does not promise trading income, model improvement, or investment performance.
