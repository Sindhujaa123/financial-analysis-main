# S&P 500 Risk Analysis

A Streamlit app that scores S&P 500 companies across seven risk dimensions using public data from SEC EDGAR filings and Yahoo Finance.

Higher scores indicate higher relative risk among the companies in the dataset. The app lets you adjust the dimension weights.

## Risk dimensions

| Dimension | Example attributes | Source |
|---|---|---|
| **Financial** | Margins, ROA/ROE, growth, Altman Z-score | SEC XBRL and Yahoo Finance |
| **Liquidity** | Current, quick, and cash ratios; free cash flow margin | SEC XBRL |
| **Debt / Leverage** | Debt-to-equity, net debt/EBITDA, interest coverage | SEC XBRL and Yahoo Finance |
| **Market** | Beta, annualized volatility, max drawdown, short interest | Yahoo Finance price history |
| **Governance** | Audit, board, compensation, shareholder rights, insider activity | Yahoo Finance and available market data |
| **Innovation** | R&D intensity and growth; capex intensity | SEC XBRL |
| **Risk Language** | Year-over-year change in 10-K Item 1A risk factors; uncertainty and litigation terms | SEC filings |

Metrics are converted to relative percentiles from 0 to 100, averaged within each dimension, then combined into a weighted composite score. Some metrics are missing for certain industries; available metrics are used in each dimension's average.

## Requirements

- Python 3.12 (see `.python-version`)
- Network access to SEC EDGAR, Yahoo Finance, and Wikipedia when refreshing the dataset
- A descriptive contact string in `SEC_USER_AGENT` before making SEC requests, as required by [SEC fair-access guidance](https://www.sec.gov/os/accessing-edgar-data)

## Setup

```bash
python -m venv .venv
```

Activate the environment, then install dependencies:

```bash
# macOS / Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt
```

Set `SEC_USER_AGENT` to a descriptive value that includes contact information before rebuilding data. For example, in PowerShell:

```powershell
$env:SEC_USER_AGENT = "SP500RiskResearch your.email@example.com"
```

## Run the app

The repository includes a processed dataset so the app can start without first downloading data.

```bash
streamlit run app.py
```

Open http://localhost:8501 in your browser.

## Build or refresh the dataset

The default build processes the first 100 companies. To process all current index constituents:

```bash
python build_dataset.py
python build_dataset.py --limit 503
```

The script caches downloaded API responses and filings in `data/raw/`, which is ignored by Git. Processed datasets are written to `data/processed/`.

## Project layout

```text
app.py                    Streamlit interface
build_dataset.py          End-to-end dataset builder
pipeline/
  universe.py             S&P 500 constituent list
  collect_yahoo.py        Market and company data from Yahoo Finance
  collect_sec_fundamentals.py  SEC XBRL fundamentals
  collect_sec_risk_text.py     10-K Item 1A risk language analysis
  features.py             Derived metrics
  scoring.py              Percentile normalization and composite score
  ui.py                   Theme, charts, and shared UI components
  sec_client.py           Rate-limited, cached SEC EDGAR client
data/processed/            Dataset used by the app
```

## Data sources

- [SEC EDGAR](https://www.sec.gov/edgar) for XBRL company facts and filings
- [Yahoo Finance](https://finance.yahoo.com/) via `yfinance`
- [Wikipedia](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies) for index constituents

The risk-language analysis uses a lightweight dictionary and Jaccard similarity approach. This project is for research and educational purposes, not investment advice.
