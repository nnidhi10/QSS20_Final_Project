# QSS20_Final_Project
Do Olympic Victories Move Markets? A Study of U.S. Medal Wins and Stock Performance

## What this repository is for

This project tests whether **U.S. Olympic performance** (especially gold medals) is associated with short-run changes in U.S. financial markets.

We build a clean “medal-day” dataset for the U.S., map medals to calendar dates using Olympic event pages (with fallbacks), align those dates to **NYSE trading days**, merge with daily market data (SPY + a small set of sentiment-exposed tickers), compute abnormal returns (CAPM / Fama–French), and run a set of standard event-study / regression analyses.

---

## Repository structure

- `data/`
  - `olympic_medals.csv` — raw medal-level dataset
  - `archive.zip` — (optional) archived inputs

- `main/`
  - `olympics_event_date_scraper.ipynb` — scrape event dates for USA gold medals + build daily medal series
  - `olympics_financial_data_analysis.ipynb` — merge medal-day data with market data + abnormal returns; exports regression panel
  - `olympics_full_analysis_medals_vs_markets.ipynb` — full analysis + exports `results_*.csv`
  - `olympics_results_visualizations_clear.ipynb` — clarity-first plots from `results_*.csv`
  - `cache_wiki/` — cached Wikipedia responses to reduce repeated requests

- `outputs/`
  - `usa_event_dates_lookup.csv` — unique USA gold events with scraped event dates
  - `usa_olympic_medals_with_event_dates.csv` — medal rows enriched with event dates
  - `usa_medal_daily_counts_and_score_ALL.csv` — daily USA medal counts + performance score (ALL games)
  - `usa_medal_daily_counts_and_score_SUMMER.csv` — daily USA medal counts + performance score (Summer only)
  - `usa_medal_daily_counts_and_score_WINTER.csv` — daily USA medal counts + performance score (Winter only)
  - `usa_medals_market_panel.csv` — (intermediate) merged panel (if generated)
  - `usa_medals_market_panel_regression_subset.csv` — regression-ready subset exported by Notebook 2

- `results/` (analysis outputs exported by the full analysis notebook)
  - `results_corr_matrix.csv`
  - `results_diff_in_means.csv`
  - `results_core_regressions_long.csv`
  - `results_sector_regressions_long.csv`
  - `results_season_interaction_long.csv`
  - `results_lead_lag_long.csv`
  - `results_car_summary.csv`

---

## Methodology (high level)

### 1) Build medal-day dataset (Notebook: `olympics_event_date_scraper.ipynb`)

**Goal:** map medal outcomes to event dates, then aggregate to a daily U.S. medal series.

Steps:
1. Read `data/olympic_medals.csv`.
2. Filter to USA (default filter is `country_3_letter_code == "USA"`) and (by default) **GOLD** medals.
3. Create a **unique event list** for scraping (deduping team medals to avoid overcounting/overscraping).
4. Scrape event date with fallbacks:
   - Method 1: Wikipedia HTML infobox `Dates`
   - Method 2: Wikidata properties (`P585`, `P582`, `P580`)
   - Method 3: Wikipedia wikitext infobox parameters (MediaWiki API)
5. Convert date ranges (e.g., `2–8 Feb 2022`) to a single date using the **end of the range** (`event_date_end`) for event-study alignment.
6. Export:
   - `outputs/usa_event_dates_lookup.csv`
   - `outputs/usa_olympic_medals_with_event_dates.csv`
   - `outputs/usa_medal_daily_counts_and_score_{ALL,SUMMER,WINTER}.csv`

### 2) Merge with market data + compute abnormal returns (Notebook: `olympics_financial_data_analysis.ipynb`)

**Goal:** create a daily panel with medals + market outcomes + abnormal returns.

Market data sources used at runtime:
- Yahoo Finance (`yfinance`) for prices/volume
- Ken French / Fama–French factors via `pandas_datareader`

Design choice:
- If a medal date falls on a non-trading day, it is shifted to the next NYSE trading day (using `pandas_market_calendars`).

Tickers used (defaults):
- Broad market: `SPY`
- Sentiment / consumer exposure examples: `NKE`, `XLY`, `DKNG`

Abnormal returns:
- CAPM and/or Fama–French 3-factor (depending on settings)

Export:
- `outputs/usa_medals_market_panel_regression_subset.csv`

### 3) Full analysis + result tables (Notebook: `olympics_full_analysis_medals_vs_markets.ipynb`)

**Goal:** rebuild a full trading-day panel (medal and non-medal days) and run the complete analysis suite.

The notebook produces:
1. Correlation matrix (medals ↔ market outcomes)
2. Difference-in-means (medal days vs non-medal days)
3. Core regressions (medal indicators, counts, performance score)
4. Sector/ticker heterogeneity (SPY vs NKE vs XLY vs DKNG)
5. Volume/volatility regressions
6. Summer vs Winter interactions
7. Lead/lag tests (k = -1, 0, +1)
8. Placebo tests (randomized medal days)
9. Event-window CAR summaries ([-1, +1], [-1, +3])

Exports to `results/`:
- `results_corr_matrix.csv`
- `results_diff_in_means.csv`
- `results_core_regressions_long.csv`
- `results_sector_regressions_long.csv`
- `results_season_interaction_long.csv`
- `results_lead_lag_long.csv`
- `results_car_summary.csv`

### 4) Clarity-first plots (Notebook: `olympics_results_visualizations_clear.ipynb`)

**Goal:** generate a small number of figures directly from `results/*.csv`.

Design choices:
- show values in percent / basis points
- symmetric axes around 0
- prefer error bars (≈95% CI) when possible
- reduce the number of plots to those that answer the main questions quickly

---

## How to run (recommended order)

1. `main/olympics_event_date_scraper.ipynb`
2. `main/olympics_financial_data_analysis.ipynb`
3. `main/olympics_full_analysis_medals_vs_markets.ipynb`
4. `main/olympics_results_visualizations_clear.ipynb`

---

## Requirements

These notebooks use (at minimum):
- pandas, numpy
- requests, beautifulsoup4, lxml, tqdm, python-dateutil, rapidfuzz
- yfinance
- pandas_datareader
- pandas_market_calendars
- statsmodels
- scipy
- matplotlib

Note: Notebooks 2 and 3 use online data sources (Yahoo Finance + Ken French factors).

---

## Outputs you should expect

After running the pipeline:
- `outputs/` will contain the cleaned medal-day datasets and a regression-ready market panel.
- `results/` will contain analysis tables used by the visualization notebook and any write-up.
