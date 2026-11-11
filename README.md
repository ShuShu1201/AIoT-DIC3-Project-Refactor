# FMPA — Fresh Market Price Prediction and Analysis Agent

FMPA is a domain-oriented AI agent prototype for estimating the 7-day and 14-day price direction of fresh-market items. It anchors each estimate to a user's **current price at an individual retail store** and transfers price-change signals learned from public market data.

> **Project status:** This repository was empty when inspected. This initial README documents the project based on the supplied project report; application source code, model artifacts, datasets, and reproducible setup scripts are not yet present in this repository.

## Overview

Consumers often know the price they are paying at a particular store, but do not have a historical price series for that store. Public market quotations provide useful market-level signals, but they are not equivalent to an individual shop's retail price.

FMPA addresses this gap by combining:

- A user-provided current store price, item, and unit.
- Daily historical quotations from 13 public markets in Taichung.
- Forecasts of price-change direction over **7 and 14 calendar days**.
- A Web/MCP agent workflow that separates language interaction from numerical forecasting.

The system is intended to provide an interpretable reference estimate, not a guaranteed future price or a reliable price-rise alert.

## System architecture

The project report describes five main modules:

1. **User entry** — Web interface and AI assistant for receiving the item, current store price, and unit.
2. **MCP tools** — get_market_quote retrieves public market quotations; post_forecast requests a forecast.
3. **Oracle API / web layer** — Validates inputs and cleans incoming fields.
4. **ForecastEngine** — Runs the 7-day Random Forest (RF) and 14-day HistGradientBoosting (HGB) forecasting models.
5. **Public historical data** — Taichung public-market historical records and daily market snapshots.

The LLM is limited to intent understanding, tool selection, and natural-language explanation. It is **not** responsible for calculating or guessing prices; numerical estimates are produced by the domain forecasting pipeline.

## Forecasting modes

Let r_h denote the estimated market price-change rate for horizon h, where h is 7 or 14 calendar days.

### Mode A — Current-price anchoring

The reference price is calculated from the user's current store price:

P_A(t+h) = P_store(t) × (1 + r_h)

This mode transfers the estimated market-level percentage change to the current store-price scale. The project report notes that this scaling changes the reference amount, not the direction classification.

### Mode B — Previous-week price adjustment

When a comparable price from the **same store and same item specification** one week earlier is available, the system applies an additional price-gap momentum adjustment. The report specifies alpha = 0.3 as a fixed, untrained assumption.

Mode B should therefore be treated as experimental. Its incremental value has not yet been established through paired validation against actual, matured store prices.

## Data and evaluation

The reported experiments use Taichung public-market data, with a dataset snapshot archived on **2026-10-02**. The evaluation uses a chronological train/validation/test split. To avoid temporal leakage, **1,386 samples with immature target labels** were removed.

Reported model settings:

- 7-day model: Random Forest with 300 trees.
- 14-day model: HistGradientBoosting with 200 iterations.

The report evaluates:

- Three-class direction prediction using macro-F1.
- Relative-change error using MAE.
- Recall for upward price movements.

### Reported test results

| Metric | 7-day FMPA | 7-day baseline | 14-day FMPA | 14-day baseline |
|---|---:|---:|---:|---:|
| Direction macro-F1 | 0.4218 | 0.4226 (momentum) | 0.4169 | 0.4306 (momentum) |

The reported test sets contain 2,085 samples for the 7-day horizon and 1,859 samples for the 14-day horizon.

The report also states that the model's MAE did not outperform the no-change baseline and that upward recall was only **10.1%**. On this evidence, FMPA should not be presented as outperforming simple baselines or as a dependable upward-price warning system.

## Domain assumptions and safeguards

The project emphasizes the following principles:

- **Price comparability first:** Prices from different stores, specifications, or units should not be compared as if they were identical.
- **Missing quotation is not zero price:** A value of 0 represents no quotation, not a price of zero currency units.
- **Temporal integrity:** Features must use information available by the prediction time; immature target labels are excluded.
- **Strong simple baselines:** Model performance is compared with no-change and momentum-persistence baselines.
- **Public data is not private store data:** Public-market trends are used as signals, not treated as direct observations of an individual shop's price.
- **Prediction error is not user benefit:** Evaluation metrics alone do not establish practical decision value.

The reported query algorithm also includes input validation, unit conversion between Taiwan catty (台斤) and kilogram, historical-feature loading, stale-quotation warnings, model estimation, median aggregation, and forecast-mode selection.

## Limitations and future work

The current reported evaluation does not demonstrate that the machine-learning models outperform the simple baselines. In particular:

- Public market quotations may not reflect store-specific pricing.
- Upward-movement recall is too low for dependable rise alerts.
- Mode B uses a fixed alpha = 0.3 assumption rather than a parameter validated on paired store-level observations.
- The key next step is to collect matured prices for the same store and same item specification, then conduct paired evaluation to determine whether Mode B adds value.

## Repository contents and reproducibility

At the time this README was created, the GitHub repository contained no files. Consequently, this repository does not yet provide:

- Application or MCP server source code.
- Forecast model training or inference scripts.
- Dataset files or model artifacts.
- Dependency manifests, environment configuration, or deployment instructions.

Those materials must be added before the prototype can be installed or reproduced from this repository. No commands are listed here as executable setup steps because the report does not establish a verified repository-specific installation procedure.

## Acknowledgement

This README is based on the supplied FMPA project report. Reported results and design descriptions are attributed to that report; implementation details not present in the repository have intentionally not been inferred.
