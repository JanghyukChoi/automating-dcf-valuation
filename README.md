# 📈 Automatic WACC + DCF Valuation Notebook

This repository contains a single Jupyter notebook (``) that

downloads the latest financial statements and daily prices for any listed ticker with yfinance,

builds a forward‑looking WACC from market data (CAPM‑based Re, statement‑derived Rd, tax shield, capital weights),

projects 5 years of Free Cash Flow using driver‑lite ratios (historical CAGR, EBIT margin, Dep/Rev, ΔNWC/Rev),

discounts those FCFs to an enterprise value (explicit 5 Y + Gordon terminal),

converts EV → equity value → intrinsic share price, and finally

benchmarks that price against the latest market close, printing the premium / discount.

All steps run end‑to‑end inside the notebook – no manual data entry is required.

1 · Quick Start

# (1) clone & enter repo
$ git clone <repo-url>
$ cd <repo>

# (2) create & activate env (optional but recommended)
$ python -m venv venv
$ source venv/bin/activate         # Windows: venv\Scripts\activate

# (3) install dependencies
(venv) $ pip install -r requirements.txt

# (4) launch Jupyter Lab / Notebook
(venv) $ jupyter lab   # or:  jupyter notebook

# open valuation.ipynb and run all cells

Command‑line execution is also supported via papermill or the Execute Notebook button in VS Code.

2 · Notebook Walk‑Through

Section

What happens

Key outputs

0 · Parameters

 Ticker symbol, market proxy, rf, ERP, look‑back & forecast horizon, terminal g.

Editable cell with sensible defaults.

1 · Market Data

yfinance pull → daily returns (stock & index)

DataFrame of percent moves

2 · β & Re

OLS regression → raw β → Blume adjustment → CAPM

Raw β, Adjusted β, Cost of Equity

3 · Rd & Tc

Statement scraper → Interest Expense ÷ Total Debt, Tax Expense ÷ EBT

Cost of Debt (pre‑tax), Effective tax rate

4 · WACC

Weighted average with tax shield

WACC summary table

5 · Historical Ratios

Build 5‑year frame (Revenue, EBIT, Tax, Dep, ΔNWC)

CAGR, EBIT‑margin, Dep/Rev, ΔNWC/Rev

6 · FCF Forecast

Geometric revenue growth, ratio‑based cost lines

5‑year projection table

7 · DCF

PV of FCF + Gordon terminal

Enterprise Value

8 · Equity Bridge

EV − Net Debt = Equity Value → ÷ shares

Intrinsic price per share

9 · Market Check

Fetch last close

Premium / (discount) %

All intermediate values (β, Re, Rd, WACC, FCFs, PV factors) are surfaced in the notebook for full transparency.

3 · Core Assumptions

Risk‑free rate (``) default ≈ 3 %; editable

Equity‑risk premium (``) default ≈ 5 %; editable

Forecast horizon 5 years; revenue grows at historical CAGR

CapEx = Depreciation (maintenance only) — conservative, quick‑and‑dirty

ΔNWC / Rev locked to historical mean

**Terminal growth **g default 2 %

Change any of these in the first code cell to re‑run scenarios instantly.

4 · Folder Structure

.
├── valuation.ipynb          # ▶ Jupyter notebook (main logic)
├── requirements.txt         # yfinance, pandas, numpy, statsmodels, etc.
└── README.md                # this file

5 · Planned Enhancements

Yield‑curve Rd – estimate cost of debt from traded bond YTM instead of I/S division.

Driver‑rich FCF – break revenue into Volume × ASP or user metrics; separate maintenance vs growth CapEx.

Monte‑Carlo wrapper – Sobol‑sequence draws for growth, margin, WACC, g → percentile valuation bands.

Quarterly cadence – switch to quarterly_financials + seasonal SARIMAX for top‑line forecasting.

Streamlit dashboard – drag‑and‑drop tickers, slider‑based assumptions, downloadable PDF tear‑sheets.

