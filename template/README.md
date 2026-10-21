# Templates

> **Public edition:** This repository contains selected methodology, templates, and synthetic examples for the Stock Analysis research framework. Private research records, account data, credentials, and operational configuration are excluded. It documents the research workflow and is not a complete application distribution.

This folder contains reusable templates consumed by scheduled and interactive research agents.

## Design Rule

Scheduled tasks should contain only:

- the template path,
- the input values required by that template.

They should not duplicate forecasting methodology, parameter definitions, research rules, output schemas, storage rules, or ticker-specific logic.

## Current Templates

### `stock_agent.md`

Generic stock-research agent wrapper.

Inputs:

- `TICKER`
- `MODE`

Supported modes:

- `intraday`
- `intraday_expiration`
- `close`

### `daily_analysis.md`

Reusable human-readable post-close analysis format.

## Stock-Specific Exceptions

Ticker-specific metadata or explicit exceptions belong under:

`stocks/<folder>/stock.json`

Stock-specific learned state should live under that stock's folder.

The default should always be shared behavior through templates and common skills; stock-specific logic is the exception.
