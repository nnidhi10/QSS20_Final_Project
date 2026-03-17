# QSS20 Final Project
## Do Olympic Victories Move Markets?
### A Cross-Country Study of Olympic Performance and Stock Market Reactions

This project investigates whether Olympic medal performance influences short-term stock market behavior.

We study three countries with very different market structures:

- **United States** — large, highly diversified financial market
- **Norway** — smaller economy with stronger national identity around Winter Olympics
- **New Zealand** — medium-size developed economy with several recent Olympic performance shocks

By comparing these countries, we test whether national sporting success generates measurable sentiment effects in financial markets.

The project builds Olympic medal datasets with precise event dates, aligns those dates to national trading calendars, merges them with daily financial market data, and runs a series of event-study and regression analyses.

---

## Repository Structure
```
QSS20_Final_Project/
│
├── data/
│   ├── raw/
│   │   └── olympic_medals.csv
│   │
│   └── interim/
│       ├── usa/
│       ├── norway/
│       └── nz/
│
├── notebooks/
│   ├── olympics_event_date_scraper.ipynb
│   ├── olympics_financial_data_analysis.ipynb
│   ├── olympics_full_analysis_medals_vs_markets.ipynb
│   ├── olympics_results_graph_builder_selected_metric.ipynb
│   ├── norway_olympics_market_event_study.ipynb
│   └── nz_olympics_market_event_study.ipynb
│
├── outputs/
│   ├── usa_*.csv
│   ├── norway_*.csv
│   └── nz_*.csv
│
├── results/
│   ├── usa_results_*.csv
│   ├── norway_results_*.csv
│   └── nz_results_*.csv
│
├── figures/
│   ├── us_presentation_figures/
│   ├── norway_presentation_figures/
│   └── nz_presentation_figures/
│
├── cache/
│   ├── cache_wiki/
│   ├── cache_wiki_norway/
│   └── cache_wiki_newzealand/
│
└── presentation/
```

### Key Folders

| Folder | Purpose |
|--------|---------|
| `data/raw` | Original Olympic medal dataset |
| `outputs` | Cleaned medal-day datasets and merged market panels |
| `results` | Statistical outputs from regressions and event studies |
| `figures` | Figures used in presentation slides |
| `notebooks` | Analysis workflow notebooks |
| `cache` | Cached Wikipedia pages used during event date scraping |

---

## Methodology

### 1. Build Olympic Medal-Day Dataset

**Notebook:** `notebooks/olympics_event_date_scraper.ipynb`

**Goal:** Map Olympic medal events to exact calendar dates.

**Steps:**
1. Load the raw Olympic medal dataset.
2. Filter to the target country:
   - USA
   - Norway
   - New Zealand
3. Construct a unique event list to avoid scraping duplicate team medals.
4. Scrape event dates using multiple fallback methods:
   - **Primary source:** Wikipedia infobox date fields
   - **Fallbacks:** Wikidata event properties, Wikipedia page wikitext
5. Convert event date ranges (e.g. 2–8 Feb 2022) into a single event date using the end of the range, which best reflects when medal outcomes are finalized.

**Outputs:**
```
outputs/{country}_event_dates_lookup.csv
outputs/{country}_olympic_medals_with_event_dates.csv
outputs/{country}_medal_daily_counts_and_score_{ALL,SUMMER,WINTER}.csv
```

These files create a daily time series of medal performance.

---

### 2. Merge Olympic Performance with Financial Markets

**Notebook:** `notebooks/olympics_financial_data_analysis.ipynb`

**Goal:** Build a trading-day panel dataset combining medals and market outcomes.

**Steps:**
1. Map medal event dates to trading days using national exchange calendars.

| Country | Trading Calendar |
|---------|-----------------|
| USA | NYSE |
| Norway | Oslo Stock Exchange |
| New Zealand | NZX |

> If a medal event occurs on a non-trading day, it is shifted to the next trading day.

2. Download market data from Yahoo Finance using `yfinance`.

**Market benchmarks:**

| Country | Market Index |
|---------|-------------|
| USA | SPY (S&P 500 ETF) |
| Norway | OSEBX |
| New Zealand | NZ50 |

3. Compute financial variables:
   - Daily returns
   - Log volume changes
   - Rolling volatility
   - Abnormal returns

> Abnormal returns are estimated using rolling CAPM market models and rolling mean models for the benchmark index.

**Outputs:**
```
outputs/{country}_medals_market_panel.csv
```

These panels contain one row per trading day.

---

### 3. Event-Study and Regression Analysis

**Notebook:** `notebooks/olympics_full_analysis_medals_vs_markets.ipynb`

This notebook performs the main statistical analysis.

**Analyses include:**

- **Correlation analysis** — Relationship between Olympic performance and financial outcomes.
- **Difference-in-means tests** — Compare medal days vs. non-medal days.
- **Core regressions** — Dependent variables include returns, abnormal returns, trading volume, and volatility. Key explanatory variables: medal indicator, medal counts, performance score.
- **Sector / firm heterogeneity** — Compare index performance to selected companies.
- **Seasonal effects** — Separate effects for Summer and Winter Olympics.
- **Lead-lag tests** — Tests whether market effects occur before, on, or after medal wins.
- **Event-window CAR** — Cumulative abnormal returns computed for windows such as `[-1, +1]`, `[0, +1]`, `[0, +3]`, `[0, +5]`.

**Outputs saved to:**
```
results/{country}_results_*.csv
```

---

### 4. Visualization and Presentation Figures

**Notebook:** `notebooks/olympics_results_graph_builder_selected_metric.ipynb`

Generates clear figures for the presentation.

**Examples:**
- Medal vs. non-medal day returns
- Performance score vs. market returns
- Lead-lag response graphs
- Seasonal effects
- Raw data graphs (medals and market returns)

**Figures are exported to:**
```
figures/{country}_presentation_figures/
```

---

## Research Questions

### Primary Question
> Do Olympic medal victories produce measurable short-term effects in national stock markets?

### Secondary Questions
Do medal wins influence:
- Stock market returns
- Trading volume
- Volatility

Are these effects stronger in smaller or less diversified markets?

### Comparative Question
> If Olympic victories do not affect large diversified markets like the U.S., do they have stronger impacts in smaller markets such as Norway and New Zealand?

---

## Expected Outputs

After running the full pipeline:

- **`outputs/`** — Clean medal datasets and market panels.
- **`results/`** — Statistical analysis tables.
- **`figures/`** — Presentation-ready figures.

---

## Requirements

Python packages used:
```
pandas
numpy
requests
beautifulsoup4
lxml
tqdm
rapidfuzz
python-dateutil
yfinance
pandas_market_calendars
statsmodels
scipy
matplotlib
```

> Several notebooks require internet access to download Wikipedia pages and Yahoo Finance data.

---

## Running the Project

Recommended order:

| Step | Notebook |
|------|---------|
| 1 | `olympics_event_date_scraper.ipynb` |
| 2 | `olympics_financial_data_analysis.ipynb` |
| 3 | `norway_olympics_market_event_study.ipynb` |
| 4 | `nz_olympics_market_event_study.ipynb` |
| 5 | `olympics_full_analysis_medals_vs_markets.ipynb` |
| 6 | `olympics_results_graph_builder_selected_metric.ipynb` |

---

## Key Insight of the Study

Early results suggest:

- 🇺🇸 **U.S. markets** show little measurable reaction to Olympic performance, likely due to their scale and diversification.
- 🇳🇴 **Smaller national markets** (e.g., Norway) display slightly stronger sentiment effects.
- 🇳🇿 **New Zealand** provides a useful test case for examining whether unexpected Olympic success can generate measurable financial responses.
