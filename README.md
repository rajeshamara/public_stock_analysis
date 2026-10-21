# Stock Analysis

> **Public edition:** This repository contains selected methodology, templates, and synthetic examples for the Stock Analysis research framework. Private research records, account data, credentials, and operational configuration are excluded. It documents the research workflow and is not a complete application distribution.

## A versioned framework for forecast research

Stock Analysis organizes a repeatable research workflow: capture the information available at a particular time, freeze a forecast, record the later outcome, and evaluate what improved or weakened the prediction.

The project separates intraday price research, option-expiration price research, and later interpretation. It makes assumptions, uncertainty, data gaps, and methodology changes explicit so that forecasts can be compared without rewriting history.

**This public edition contains research protocols, JSON record examples, and reusable agent instructions.** It does not include an application runtime, data-provider integration, scheduler, trained forecasting model, or brokerage connection. The examples contain fictional identifiers and unpopulated measurements. No historical performance or investment return is claimed.

## Framework capabilities

| Design area | Implementation in this repository |
|---|---|
| Research workflow | Separate forecast, outcome, daily rollup, and interpretive analysis records |
| Time-aware data modeling | Forecast timestamps, information cutoffs, immutable checkpoints, and source summaries |
| Consistent inputs | A registry of 100 named parameters covering price, volatility, options context, market conditions, events, news, and fundamentals |
| Reusable agent design | One stock-agent template with explicit ticker resolution, processing modes, and isolation rules |
| Evaluation planning | Walk-forward comparisons, forecast error, interval coverage, probability calibration, and revision analysis |
| Reproducibility | Separate schema, methodology, and model identifiers; missing values remain explicit |

The statistical techniques described in the protocols are evaluation plans. This repository does not contain an implemented analytics engine or evidence that the hypotheses produce a predictive advantage.

## Research workflows

### Intraday checkpoints

The intraday protocol records how expectations change as information arrives. Before the close, each checkpoint tracks six targets: the current day's close, high, and low, and the next trading day's close, high, and low. The documented cadence includes a 10:45 ET checkpoint to test a timing hypothesis rather than assume it is useful.

After the close, a rollup compares frozen predictions with observed outcomes. A separate Markdown analysis explains revisions, uncertainty, unexpected events, and questions for later testing.

### Expiration-price forecasts

The expiration protocol records a distribution for the underlying stock's closing price at a selected expiration. It includes central estimates, probability intervals, level probabilities, and an information cutoff. Forecasts and outcomes remain separate records. Selecting trades or executing orders is outside the framework.

## Repository guide

| Path | Purpose |
|---|---|
| [RESEARCH_CHARTER.md](RESEARCH_CHARTER.md) | Research goals, evidence standards, and scope |
| [FORECAST_PROTOCOL.md](FORECAST_PROTOCOL.md) | Expiration forecast and outcome lifecycle |
| [INTRADAY_FORECAST_PROTOCOL.md](INTRADAY_FORECAST_PROTOCOL.md) | Checkpoints, six forecast targets, revisions, and post-close evaluation |
| [PARAMETER_REGISTRY.md](PARAMETER_REGISTRY.md) | Stable parameter IDs and measurement conventions |
| [DATA_LAYOUT.md](DATA_LAYOUT.md) | Local configuration and record organization |
| [template/stock_agent.md](template/stock_agent.md) | Generic agent entry point with `TICKER` and `MODE` inputs |
| [template/daily_analysis.md](template/daily_analysis.md) | Post-close research journal template |
| [skills/stock-forecast/SKILL.md](skills/stock-forecast/SKILL.md) | Shared agent operating procedure |
| [schemas/](schemas/) | Example JSON record shapes |
| [examples/](examples/) | Synthetic, inactive configuration examples |

## Getting started

An editor is sufficient to read and adapt the framework. Python 3 is optional for checking JSON syntax. To execute the agent instructions, supply your own agent runtime and authorized sources for the data you intend to use. No credentials are needed to inspect this repository.

1. Read the charter and choose the research question, symbols, horizons, and data sources for your experiment.
2. Create local configuration from the synthetic examples:

   ```bash
   mkdir -p stocks/EXAMPLE
   cp examples/stock-index.example.json stocks/index.json
   cp examples/stock.example.json stocks/EXAMPLE/stock.json
   ```

3. Replace the fictional `EXAMPLE` symbol and company with your own research configuration. Keep `ticker` and `folder` consistent between the index, metadata, and directory. Set `example_only` to `false` for genuine local configuration. Leave `research_status` as `inactive` until configuration and authorized data sources are ready; set it to `active` deliberately when you want the template to process that symbol.
4. Provide `template/stock_agent.md` to your chosen agent with a configured `TICKER` and one of `intraday`, `intraday_expiration`, or `close`. Resolve repository paths relative to this checkout. These are prompt templates; they do not install or start a service.
5. Keep generated research under the local `stocks/` directory, which is ignored by Git. Record sources and timestamps, preserve unavailable measurements as `null`, and keep forecasts frozen when outcomes arrive.

The supplied configuration is inactive by design. The template stops for an unknown or inactive symbol. No schedule is installed by these steps, and the repository supplies no credentials or order-placement capability.

### Inspect a record shape

For example, check the current intraday record's JSON syntax with:

```bash
python3 -m json.tool schemas/intraday_checkpoint_v1.example.json
```

The command checks JSON syntax only. The files in `schemas/` are example instances, not formal JSON Schema documents or executable validators. Null measurements, zero placeholder counts, and `example_only: true` mark examples; they are not completed forecasts.

### Format versions

| Example family | Version | Intended role |
|---|---|---|
| `intraday_checkpoint`, `daily_rollup` | 1.1 | Current checkpoint workflow described in the intraday protocol |
| `forecast_record`, `outcome_record` | 1.0 | Expiration forecasts and their later outcomes |
| `daily_forecast`, `daily_outcome` | 1.0 | Earlier daily-only format retained for reference |

The earlier daily examples use `daily/forecasts/`; the current intraday workflow uses `daily/checkpoints/`. Treat these as distinct formats. There is no automatic migration or compatibility layer. Protocol versions and record-format versions are separate concepts.

## Public edition and local data

This edition publishes the reusable methodology and templates. Historical forecasts, stock-specific research, account or position information, personal performance records, provider credentials, and private infrastructure are excluded. All published JSON examples use the fictional symbol `EXAMPLE`; generic benchmark symbols appearing in explanatory text are illustrative mappings.

Keep provider credentials in the facilities supported by your chosen runtime, outside committed files. The supplied `.gitignore` excludes local research records, environment files, common credential files, logs, and generated exports. Review any additional files before publishing them.

## Current limits and next steps

The framework specifies how research should be recorded and evaluated. Data acquisition, market-calendar handling, interval estimation, numerical scoring, scheduling, and schema validation require a separately implemented runtime or a carefully supervised manual workflow. Potential next steps include formal JSON Schemas, an offline evaluator, simple baseline comparisons, and reproducible walk-forward reports.

The research is allowed to conclude that a hypothesis has no durable predictive value. Forecast quality must be measured from suitable data; the design of the workflow alone does not establish accuracy or profitability.
