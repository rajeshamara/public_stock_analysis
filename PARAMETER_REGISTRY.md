# Parameter Registry

Version: `0.1`

This document defines the stable parameter vocabulary used by the stock-expiration forecasting experiment.

The purpose of the registry is to prevent the experiment from drifting into ad-hoc reasoning. Every forecast may use many inputs, but those inputs should have consistent names, meanings, units, and timestamps so that results can be compared across configured symbols and observation periods.

Ticker and benchmark mappings in this document are illustrative examples, not an active watchlist or recommendations. Configure and document appropriate mappings for the actual research universe.

The registry is intentionally broad. Some parameters may later prove useless. Others may matter strongly for one ticker and barely matter for another. The experiment must discover that from data rather than assuming it in advance.

---

## Core Rules

1. **Stable IDs**  
   Once an ID such as `P34` has been assigned, its meaning must not silently change.

2. **No future information**  
   A forecast may contain only information that was available at the forecast timestamp.

3. **Missing is different from zero**  
   If a parameter cannot be obtained reliably, record it as JSON `null` (or `NA` in narrative notes) rather than inventing or substituting a value.

4. **Raw value first**  
   Preserve the raw measured value when possible. Derived normalized versions may be added later under new parameter IDs.

5. **Ticker-specific importance is learned, not assumed**  
   The same master registry is used for every ticker even though feature importance may differ substantially.

6. **Source and timestamp should be retained**  
   Any automated implementation should store the source and observation timestamp for each externally retrieved value.

7. **News duplication must be controlled**  
   Ten articles repeating one original analyst note are one underlying event, not ten independent signals.

8. **Parameters may be retired but not reused**  
   If a parameter proves unreliable, mark it retired. Do not give its ID a new meaning.

---

## Value Conventions

Unless otherwise noted:

- returns and changes are percentages,
- price distances should preferably be stored both in dollars and as a percentage in the eventual data model,
- volatility values are annualized percentages when supplied that way by the source,
- intraday measurements are calculated only using information available up to the forecast timestamp,
- Eastern Time is the canonical trading-session timezone,
- official regular-session close is the primary expiration target unless the protocol explicitly changes.

---

# P1-P10 — Current Stock State

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P1 | Spot price | Underlying price at the exact forecast timestamp. |
| P2 | Previous close | Official prior regular-session close. |
| P3 | Today open | Official/current regular-session opening price. |
| P4 | Opening gap % | `(P3 - P2) / P2 * 100`. |
| P5 | Return since open % | `(P1 - P3) / P3 * 100`. |
| P6 | Return since previous close % | `(P1 - P2) / P2 * 100`. |
| P7 | Distance from intraday high % | Percentage distance between current spot and session high so far. |
| P8 | Distance from intraday low % | Percentage distance between current spot and session low so far. |
| P9 | Distance from VWAP % | Current price relative to session VWAP. |
| P10 | Premarket return % | Last premarket price versus prior close, measured before regular-session open. |

---

# P11-P20 — Momentum and Realized Volatility

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P11 | Previous after-hours return % | Prior post-market move from regular close to final/reliable after-hours observation. |
| P12 | 5-minute return % | Return over the latest completed 5-minute interval. |
| P13 | 15-minute return % | Return over the latest 15 minutes. |
| P14 | 30-minute return % | Return over the latest 30 minutes. |
| P15 | 60-minute return % | Return over the latest 60 minutes. |
| P16 | 5-trading-day return % | Current spot versus close five trading sessions earlier. |
| P17 | 20-trading-day return % | Current spot versus close twenty trading sessions earlier. |
| P18 | ATR-14 % | 14-session Average True Range divided by current spot. |
| P19 | 5-day realized volatility | Realized volatility computed from the previous five completed sessions. |
| P20 | 20-day realized volatility | Realized volatility computed from the previous twenty completed sessions. |

---

# P21-P30 — Volume, Range, and Technical State

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P21 | Relative volume by time of day | Cumulative volume up to forecast time divided by typical cumulative volume at the same time of day. |
| P22 | Relative dollar volume | Current cumulative dollar volume versus normal same-time dollar volume. |
| P23 | Intraday range ratio | Session high-low range so far divided by typical comparable intraday range. |
| P24 | RSI-14 | Standard 14-period RSI calculated using completed daily bars unless a later protocol specifies otherwise. |
| P25 | Distance from 20-day moving average % | Spot relative to 20-session simple moving average. |
| P26 | Distance from 50-day moving average % | Spot relative to 50-session simple moving average. |
| P27 | Distance from 200-day moving average % | Spot relative to 200-session simple moving average. |
| P28 | Distance from 20-day high % | Spot relative to highest completed-session high over prior 20 trading days. |
| P29 | Distance from 20-day low % | Spot relative to lowest completed-session low over prior 20 trading days. |
| P30 | Opening-range state | Encoded state: below opening range / inside / above opening range. Opening-range duration must be recorded, initially 30 minutes. |

---

# P31-P40 — Options-Implied Information

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P31 | ATM implied volatility | Near-ATM IV for the target expiration, or nearest reliable interpolated value. |
| P32 | IV rank | Current IV relative to its high-low range over the chosen trailing period; initial convention: one year. |
| P33 | IV percentile | Percent of historical IV observations below current IV over the chosen trailing period; initial convention: one year. |
| P34 | Implied expected move to expiration | Expected absolute move through target expiration derived consistently from option pricing. Exact formula/source must be recorded. |
| P35 | ATM straddle % of spot | Cost of nearest ATM call + put for target expiry divided by underlying spot. |
| P36 | 25-delta put IV | Implied volatility near the 25-delta put for target expiry. |
| P37 | 25-delta call IV | Implied volatility near the 25-delta call for target expiry. |
| P38 | Put-call volatility skew | `P36 - P37`, in volatility points. |
| P39 | Near-term IV term structure | Target-expiry ATM IV relative to the next meaningful later expiration. |
| P40 | Put/call option-volume ratio | Put volume divided by call volume for the underlying using data available by forecast time. |

---

# P41-P50 — Options Positioning and Market Structure

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P41 | Put/call open-interest ratio | Total relevant put OI divided by call OI. Source and expiration scope must be recorded. |
| P42 | Option volume relative to normal | Current aggregate option volume versus a trailing same-time baseline when available. |
| P43 | Distance to largest nearby put-OI strike | Percentage/dollar distance from spot to the nearest materially dominant put open-interest strike. |
| P44 | Distance to largest nearby call-OI strike | Percentage/dollar distance from spot to the nearest materially dominant call open-interest strike. |
| P45 | ATM gamma | Gamma of nearest ATM option for target expiry, using a consistent source. |
| P46 | ATM theta | Theta of nearest ATM option for target expiry, using a consistent source. |
| P47 | ATM vega | Vega of nearest ATM option for target expiry, using a consistent source. |
| P48 | 0-DTE share of option volume | Percentage of same-day-expiration volume out of total option volume when available. |
| P49 | Distance from max-pain level | Spot relative to independently calculated/reliably sourced max-pain level. Experimental parameter; may be retired if not useful. |
| P50 | Dealer gamma / GEX regime | Reliable gamma-exposure measure or categorical positive/neutral/negative regime when a defensible source exists; otherwise `NA`. |

---

# P51-P60 — Broad Market, Sector, and Volatility Context

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P51 | SPY return % | SPY move from previous close to forecast timestamp. |
| P52 | QQQ return % | QQQ move from previous close to forecast timestamp. |
| P53 | Relevant sector ETF return % | Ticker-specific sector benchmark at forecast time. Initial mapping may use XLK for AAPL/MSFT, XLC for GOOGL/META, and XLF for BRK/B; mapping is documented and may evolve under version control. |
| P54 | Peer-basket return % | Equal-weight or documented weighted return of a fixed ticker-specific peer basket. Basket composition must be versioned separately. |
| P55 | Mega-cap basket ex-target % | Return of a fixed mega-cap basket excluding the target stock. |
| P56 | S&P 500 breadth | Percentage of S&P 500 constituents advancing, or a consistent market-breadth equivalent. |
| P57 | Nasdaq breadth | Percentage of Nasdaq constituents advancing, or a consistent breadth equivalent. |
| P58 | VIX level | CBOE VIX at forecast time. |
| P59 | VIX change % | VIX change from previous close. |
| P60 | VXN level | Nasdaq-100 volatility index at forecast time. |

---

# P61-P70 — Rates, FX, Commodities, and Credit

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P61 | VIX9D level | CBOE 9-day volatility index, useful for very short horizon event risk. |
| P62 | US 2-year Treasury yield | Yield level at forecast time. |
| P63 | 2-year yield intraday change | Change in basis points from prior close/reference level. |
| P64 | US 10-year Treasury yield | Yield level at forecast time. |
| P65 | 10-year yield intraday change | Change in basis points from prior close/reference level. |
| P66 | 10Y-2Y spread | `P64 - P62`, expressed in basis points. |
| P67 | US dollar index move % | DXY or consistent USD proxy move from prior close. |
| P68 | Crude oil move % | WTI front-month or consistent proxy return. |
| P69 | Gold move % | Spot/futures or consistent gold proxy return. |
| P70 | Credit/fixed-income stress | A documented liquid credit-spread or credit-ETF stress proxy. Initial implementation may be `NA` until a reliable standardized source is selected. |

---

# P71-P80 — Calendar, Time, and Scheduled Event Structure

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P71 | Weekday | Monday through Friday encoded categorically. |
| P72 | Forecast clock time | Exact Eastern Time of forecast freeze. |
| P73 | Hours to expiration close | Decimal hours from forecast timestamp to official target expiration close, excluding closed-market hours only if the implementation explicitly defines that convention. |
| P74 | Trading DTE | Number of trading sessions remaining including/through expiration according to protocol. |
| P75 | Standard monthly OpEx flag | `1` when target expiration is standard monthly options expiration, otherwise `0`. |
| P76 | Month-end flag | Whether forecast/expiration occurs within the defined month-end window. Initial definition: final trading day. |
| P77 | Quarter-end flag | Whether forecast/expiration occurs on final trading day of a calendar quarter. |
| P78 | Holiday/short-session flag | Holiday-adjacent or shortened US equity session. |
| P79 | Major macro release before expiration | Structured event indicator plus event identity for CPI, PPI, payrolls, GDP, retail sales, PCE, etc. |
| P80 | Fed event before expiration | Indicator and identity for FOMC decision, Powell speech, minutes, or other material Federal Reserve event. |

---

# P81-P90 — News, Analyst, Regulatory, and External Event Signals

These parameters require disciplined de-duplication. The unit of analysis is the underlying event, not the number of websites repeating it.

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P81 | Independent company-news event count | Count of distinct material or potentially material company-specific events known by forecast time. |
| P82 | Recency-weighted company-news sentiment | Aggregate score from `-1.0` strongly negative to `+1.0` strongly positive, with newer independent events weighted more heavily. |
| P83 | News materiality | `0-5` assessment: 0 none, 1 trivial, 2 minor, 3 meaningful, 4 major, 5 potentially thesis-changing/market-moving. |
| P84 | News novelty | `0-5` score measuring how new the information is versus already-known/repeated narrative. |
| P85 | News surprise direction/magnitude | `-5` major negative surprise through `+5` major positive surprise; `0` expected/neutral/no surprise. |
| P86 | Analyst-action score | Structured aggregate of upgrades, downgrades, initiations, estimate changes, and meaningful recommendation changes known by forecast time. Raw actions should also be retained. |
| P87 | Net price-target revision % | Weighted or median percentage change in newly published analyst price targets during a defined recent window. Raw analyst events retained. |
| P88 | Legal/regulatory event score | `-5` severe adverse to `+5` strongly favorable, with `0` no material event. Event description required when nonzero. |
| P89 | Corporate strategic/capital event score | `-5` strongly adverse to `+5` strongly favorable for material product, M&A, buyback, capital-return, partnership, management, or strategy developments. |
| P90 | External/geopolitical shock score | `0-5` severity of a broad external shock relevant before expiration; direction is captured separately in notes or a later dedicated parameter if needed. |

---

# P91-P100 — Company and Structural Context

| ID | Parameter | Definition / Measurement |
|---|---|---|
| P91 | Trading days from earnings | Signed trading-day distance from forecast date to next scheduled earnings; if earnings already occurred very recently, retain separate note and later add a post-earnings parameter if useful. |
| P92 | Forward P/E | Most recent defensible consensus forward P/E available before forecast time. |
| P93 | Valuation z-score / percentile | Current valuation relative to the company's own trailing historical valuation distribution; exact measure must be stable once implemented. |
| P94 | EPS estimate-revision signal | Recent net direction/magnitude of consensus EPS estimate revisions. |
| P95 | Revenue estimate-revision signal | Recent net direction/magnitude of consensus revenue estimate revisions. |
| P96 | Short interest | Most recently published short interest as % of float/shares; publication lag must be retained. |
| P97 | Earnings-yield minus 10Y yield | Forward earnings yield less P64, when forward P/E is meaningful. |
| P98 | Rolling beta to QQQ | Trailing beta of target daily returns to QQQ over a documented window. Initial candidate: 60 sessions. |
| P99 | Rolling correlation with QQQ | Trailing daily-return correlation with QQQ over the same documented window as P98 unless otherwise versioned. |
| P100 | Price/news confirmation score | `-1` price/volume meaningfully contradicts current news narrative, `0` unclear/mixed/no meaningful narrative, `+1` price/volume confirms narrative. Rationale required. |

---

# Ticker-Specific Mappings

The parameter IDs remain common, but some parameters intentionally use ticker-specific mappings.

## Initial P53 Sector Benchmark Mapping

| Ticker | Initial benchmark |
|---|---|
| AAPL | XLK |
| MSFT | XLK |
| GOOGL / GOOG | XLC |
| META | XLC |
| BRK/B | XLF |

This is a starting convention, not a claim that these ETFs are optimal predictors. Changes must be versioned rather than silently substituted.

## P54 Peer Basket

Peer baskets are intentionally **not yet frozen**. They should be defined in a later document after considering whether the basket should represent:

- direct business peers,
- mega-cap valuation peers,
- same-sector peers,
- or a data-driven correlation basket.

Until a basket is frozen for a ticker, P54 should be `NA` rather than improvised differently on each forecast.

---

# Candidate Future Parameters P101+

The registry is expected to grow. Candidate additions include:

- current move percentile versus 20-year history,
- current gap percentile,
- current intraday-range percentile,
- current volatility percentile,
- historical behavior for the same weekday,
- historical behavior under similar QQQ/VIX/yield regimes,
- nearest historical analog distance,
- distance from major support/resistance levels,
- order-flow imbalance,
- market-on-close imbalance,
- futures movement before open,
- overnight index futures regime,
- same-day economic surprise indices,
- news-source credibility score,
- social-media sentiment where defensible,
- post-earnings drift state,
- buyback blackout/window state,
- index rebalancing effects,
- ETF creation/redemption pressure,
- realized correlation regime,
- cross-sectional mega-cap leadership,
- and regime-cluster identifiers learned from historical data.

No candidate receives an ID until its definition is precise enough to remain stable.

---

# Parameter Quality States

Every stored parameter should eventually carry a quality state:

| State | Meaning |
|---|---|
| `OBSERVED` | Directly obtained from a reliable source at/near forecast time. |
| `DERIVED` | Calculated deterministically from observed values. |
| `ASSESSED` | Requires model/human interpretation, such as news materiality. |
| `STALE` | Latest available value is known to be delayed; timestamp retained. |
| `NA` | Not available or not reliably measurable. |

This is especially important because a stale fundamental measure and a live spot price should not be treated as equally current observations.

---

# Change-Control Rule

When the registry changes:

1. increment the version,
2. describe the change in Git history,
3. never reinterpret prior stored values under a new definition,
4. preserve backward compatibility where practical,
5. and record the registry version used by every forecast.

The goal is that a forecast made years later can still be reconstructed using the definitions that existed on the day it was made.
