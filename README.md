# A Trade War Story
### Global Trade War 2018-Present: A Power BI Analysis

A 7-page Power BI dashboard analyzing the economic, market, and policy impact of the US-China trade war and its ripple effects across equities, currencies, sectors, and inflation from 2018 through 2026.

---

## Overview

This project traces the trade war from its first tariffs in March 2018 through the renewed escalation of 2024-2025, connecting tariff policy directly to its measurable downstream effects: equity market drawdowns, currency depreciation, sector-level winners and losers, inflation spikes, and shifting trade balances. Built entirely from raw data using a star schema data model, custom DAX measures, and Power Query transformations. No black-box AI summarization is involved; every number is traceable back to source data.

## Dashboard Pages

1. **Executive Summary**: headline KPIs and key findings across the whole project
2. **Tariff Timeline & Escalation**: chronological bubble chart of all 70 tariff events, filterable by country, type, sector, and era
3. **Markets & Equities Impact**: S&P 500, Nasdaq, and Shanghai Composite performance, volatility, drawdown, and return distribution
4. **Sector Analysis**: year-by-year sector returns, sector ranking over time, and risk-vs-return positioning
5. **Trade & Macroeconomic Impact**: trade balance, exports/imports, CPI inflation, and trade volume growth
6. **Currency Impact**: USD/CNY exchange rate, volatility, and rolling correlation with US equities
7. **News & Policy Intelligence**: sentiment analysis of trade war news headlines

## Key Findings

- US-China tariffs peaked at 145% in April 2025, among the most severe escalations in modern trade history.
- The Shanghai Composite has consistently underperformed the S&P 500 since 2018, with deeper and more frequent drawdowns throughout both trade war periods.
- The US trade deficit reached its widest point in the dataset in 2025, continuing a multi-year widening trend.
- CPI inflation peaked at 9.0% in 2022, consistent with the global post-pandemic inflation surge.
- Despite escalating tariffs, total US trade volume still grew at roughly a 4% CAGR from 2018-2024. Disruption reshaped the composition of trade more than it reversed its overall growth.

## Data Sources

Built from two Kaggle datasets (11 tables total): tariff timeline and rates, stock market reaction, sector-level equity performance, currency exchange rates, trade volume and balance, CPI inflation, and tariff-related news headlines.

- [Global Trade War 2018-Present: Tariff and Markets](https://www.kaggle.com/datasets/belbino/global-trade-war-2018-present-tariff-and-markets/data)
- [US Tariff and Trade War Impact Dataset 2018-Present](https://www.kaggle.com/datasets/belbino/us-tariff-and-trade-war-impact-dataset-2018-present?resource=download)

## Tools & Techniques

- **Power BI Desktop**: report design, data modeling, DAX
- **DAX**: custom measures including rolling 30-day Pearson correlation (computed natively in DAX, no external statistical tool), running peak/drawdown calculations, dynamic RANKX-based sector rankings, and sort-by-column implementations for correctly ordered categorical bins
- **Power Query**: data cleaning, country code standardization, era tagging, chronological index workarounds for scatter chart axes
- **Data modeling**: star schema with fact and dimension tables

## Notable Technical Challenges

- **Rolling correlation in pure DAX**: a 30-day rolling Pearson correlation between currency movements and equity returns, recalculated at every point on the timeline, built entirely with native DAX (`ADDCOLUMNS`, windowed `FILTER`, and the manual correlation formula) rather than an external Python/R script.
- **Cross-table date matching without relationships**: several measures join currency and equity data by date using explicit filter expressions rather than relying on modeled relationships, since the two tables don't share a natural join key at matching grain.
- **Country code standardization**: tariff data mixed abbreviated codes and full country names inconsistently (e.g. "CHN" and "China" as separate values); resolved with a DAX-based normalization layer.

## Data Limitations

In the interest of transparency:
- News sentiment data covers April-June 2026 only (about 11 weeks). The "Sentiment by Week" chart reflects this real window rather than a multi-year trend.
- A small number of figures differ from early-stage design mockups, since the final dashboard is built on and verified against actual source data rather than placeholder estimates.
- The currency dataset doesn't include an EU27 or "World" aggregate; the country-level inflation comparison uses real individual countries (USA, China, Germany, Japan) instead.

## Author

**Rifhath Aslam Jageer Hussain**
Data & System Administrator (Analytics & Reporting), DHL Supply Chain UK
MSc Data Analytics, University of Warwick

## Data Attribution

Source datasets from Kaggle, by [belbino](https://www.kaggle.com/belbino):
- [Global Trade War 2018-Present: Tariff and Markets](https://www.kaggle.com/datasets/belbino/global-trade-war-2018-present-tariff-and-markets/data)
- [US Tariff and Trade War Impact Dataset 2018-Present](https://www.kaggle.com/datasets/belbino/us-tariff-and-trade-war-impact-dataset-2018-present?resource=download)

This project is for portfolio and educational purposes.

---

## Data Model

This section, along with Power Query Transformations and the DAX Reference below, serves as the project's methodology documentation in place of a dedicated in-dashboard page.

**Tables (11 source tables):**
`tariff_timeline`, `stock_market_reaction`, `sector_impact`, `currency_impact`, `trade_balance`, `trade_volume_annual`, `inflation_response`, `tariff_news_headlines`, `market_reaction`, `tariff_rates`, `column_metadata`.

**Relationships:**
- `trade_balance[year]` and `trade_volume_annual[year]`: **Many-to-many, both directions**. Neither table has a unique year column (trade_balance is monthly grain, trade_volume_annual has multiple rows per year across countries/indicators), so many-to-many is the correct cardinality here, not a workaround.
- Currency-to-equity correlation measures (`currency_impact` and `stock_market_reaction`) deliberately don't use a modeled relationship. Instead, they join by explicit date-equality filters inside DAX (`CALCULATE(..., table[date] = d)`), since the two tables don't share a clean relationship key at matching grain, and a real relationship would have caused ambiguous filter propagation across the rest of the model.

## Power Query Transformations

- Sorted `tariff_timeline` by date ascending, then added an **Index column** (Add Column -> Index Column -> From 1) to work around a chronological-ordering limitation when plotting scatter/bubble charts. Using the raw date field directly on a categorical X-axis produced incorrect ordering.
- Standardized column data types across date, numeric, and text fields.

## DAX Reference

Every custom measure and calculated column, grouped by the table it lives on.

### `tariff_timeline`
| Name | Type | Purpose |
|---|---|---|
| `era` | Column | Tags each tariff event into Baseline / Trade War 1.0 / Recovery / Trade War 2.0 based on date, with numeric prefixes to force correct sort order |
| `imposing_country_clean` | Column | Normalizes inconsistent country codes/names (e.g. "CHN" and "China" into one value); excludes non-country placeholders like "ALL"/"Global" |
| `target_country_clean` | Column | Same normalization, applied to the target-country field |
| `Total Tariff Events` | Measure | `COUNTROWS` of all tariff events |
| `Trade Value Affected` | Measure | Sum of estimated trade value, converted billions to trillions |
| `Avg Tariff Rate` | Measure | Average tariff rate across all events |
| `Countries Involved` | Measure | Distinct count of countries across both imposing and target roles, using the cleaned country columns, combined via `UNION` |

### `stock_market_reaction`
| Name | Type | Purpose |
|---|---|---|
| `Running Peak (Indexed)` | Column | Per-index, per-date running maximum of `indexed_to_100`, the historical high-water mark as of that date |
| `Drawdown %` | Measure | Current value vs. running peak, as a percentage decline |
| `Peak Drawdown` | Measure | The single worst `Drawdown %` across all dates, per index |
| `Best Recovery %` | Measure | Peak-to-trough recovery gain within a specific date window (Mar-Jul 2020) |
| `return_bin` / `return_bin_sort` | Columns | Bucket daily returns into 1%-wide bins for the distribution histogram, with a separate hidden sort column so bin labels display cleanly without numeric prefixes |
| `SP Daily Return` / `SSE Daily Return` | Measures | Average daily return, filtered to a specific index; building blocks for the cross-index correlation measure |
| `Correlation SP vs SSE` | Measure | Full Pearson correlation coefficient between S&P 500 and Shanghai Composite daily returns, computed manually in DAX (no native `CORREL` function exists) |
| `Widest Single-Day Swing` | Measure | Largest absolute single-day return, filtered to S&P 500; verified against source data as a real -11.98% drop on 16 March 2020, during the COVID market crash |

### `sector_impact`
| Name | Type | Purpose |
|---|---|---|
| `year` | Column | Extracted from date, for year-over-year sector comparisons |
| `Avg Annual Return by Sector` | Measure | Average daily return annualized (x252 trading days) |
| `Avg Volatility by Sector` | Measure | Average of the pre-computed 10-day volatility field |
| `Best Performing Sector` / `Best Performing Sector Return` | Measures | Dynamically finds the top sector by annualized return, and its value |
| `Worst Performing Sector` / `Worst Performing Sector Return` | Measures | Same logic, inverted |
| `Highest Volatility Sector` / `Highest Volatility Value` | Measures | Same dynamic-lookup pattern, applied to volatility |
| `Sector Rank` | Measure | `RANKX` of sectors by return within each year, feeding the sector ranking over time line chart |

### `trade_balance`
| Name | Type | Purpose |
|---|---|---|
| `year` | Column | Extracted from date, to relate monthly data to annual trade volume figures |
| `Trade Balance (Annual, Billions)` | Measure | Sum of monthly balance, corrected from the source data's actual millions-scale to true billions |
| `Largest Trade Deficit` / `Largest Trade Deficit Year` | Measures | Worst annual (not monthly) trade balance, and the year it occurred |
| `Avg Trade Balance` | Measure | Average of annual trade balance figures |

### `trade_volume_annual`
| Name | Type | Purpose |
|---|---|---|
| `Highest Export Growth` | Measure | Peak year-over-year export growth, USA |
| `Exports (USD)` / `Imports (USD)` | Measures | Filtered sums by indicator type, for the trade balance combo chart |
| `Total Trade Volume` | Measure | Combined exports plus imports, USA |
| `Trade Volume CAGR` | Measure | Compound annual growth rate of total trade volume, 2018-2024 |
| `Trade Volume Indexed` | Measure | Total trade volume rebased to 100 at 2018 |

### `inflation_response`
| Name | Type | Purpose |
|---|---|---|
| `Highest Inflation` | Measure | Peak YoY CPI inflation reading |

### `currency_impact`
| Name | Type | Purpose |
|---|---|---|
| `USD/CNY Latest` / `USD/CNY Latest Date` | Measures | Most recent exchange rate and its date, dynamically resolved rather than hardcoded |
| `Max 30-Day Move` | Measure | Largest 30-day percentage move, CNY |
| `Current 7D Volatility` | Measure | Latest 7-day rolling volatility reading |
| `Annual Depreciation` | Measure | CNY depreciation from 2018 baseline to the most recent rate |
| `FX Daily Change (CNY)` | Measure | Average daily percentage change, filtered to CNY, a building block reused across two correlation measures |
| `Correlation USDCNY vs SP` | Measure | Full-period Pearson correlation between CNY daily moves and S&P daily returns, matched across tables by explicit date filtering (no relationship) |
| `Rolling Correlation 30D` | Measure | The same correlation logic, recalculated on a trailing 30-day window at every date. The most computationally complex measure in the project |

### `tariff_news_headlines`
| Name | Type | Purpose |
|---|---|---|
| `week_start` | Column | Rounds each headline's date back to the Monday of its week, for weekly sentiment aggregation |
