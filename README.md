1 · Project Purpose
This repository combines two self-contained, audit-friendly building blocks:

File	Role
wacc_calc.py	Computes a forward-looking WACC from market prices & latest statements.
dcf_intrinsic_value.py	Pulls the last 5Y financials, projects 5 Y FCF, discounts them with the WACC from wacc_calc.py, converts EV→equity value→intrinsic price, and benchmarks it to the spot market quote.

The result is a one-command valuation stack that updates straight from Yahoo Finance without manual inputs.

2 · Quick Start
bash
복사
git clone <repo>
cd <repo>
python -m venv venv && source venv/bin/activate        # Win: venv\Scripts\activate
pip install -r requirements.txt                        # yfinance pandas numpy statsmodels

#— Run end-to-end valuation
python dcf_intrinsic_value.py --ticker AAPL --market ^GSPC
Output (abridged):

yaml
복사
===== WACC SNAPSHOT (AAPL) =====
Raw β: 1.26   Adjusted β: 1.17   Re: 8.8 %
Rd: 3.1 %     Tc: 14.9 %        WACC: 7.4 %

===== 5-YEAR PROJECTION =====
        Revenue           FCF
Y1   418 B          105 B
⋯
===== VALUATION SUMMARY =====
Enterprise Value : 3 065 B USD
Equity Value     : 3 310 B USD
Intrinsic Price  : 214.2 USD
Last Close       : 187.4 USD
Premium/(Disc.)  : 14.3 %
3 · Code Flow in Detail
3.1 wacc_calc.py
Step	Logic
Market data pull	5 years of daily closes for the stock and a supplied market proxy (default ^GSPC, KOSPI example 000660.KS).
Raw β	OLS of stock ΔP/P on market ΔP/P.
Blume adjusted β	β_adj = ⅔ β_raw + ⅓.
Re (CAPM)	rf + β_adj × ERP (defaults rf 3 %, ERP 5 %).
Rd	Latest interest expense ÷ total debt (auto-label search).
Tax rate	Income-tax-expense ÷ EBT.
Capital weights	Debt & market-cap from the same statement block.
WACC	E/V·Re + D/V·Rd·(1-Tc) returned as a dict with all sub-components.

3.2 dcf_intrinsic_value.py
Imports WACC – from wacc_calc import calculate_wacc

Financial rows pulled via label-agnostic helper grab_row().

Ratios – CAGR, EBIT-margin, dep/Rev, ΔNWC/Rev.

5-year forecast – revenue grows geometrically; other lines scale by averages.

DCF – explicit 5 Y PV + terminal Gordon (default g = 2 %).

Net debt & share count – balance-sheet & sharesOutstanding.

Intrinsic price vs last close, premium / discount %.

4 · Key Assumptions (and Where to Edit)
Item	Default	Config
Risk-free rate	3 %	--rf 0.035 (CLI)
ERP	5 %	--erp 0.045
Look-back for β	5 calendar years	--years 3
Forecast horizon	5 Y	YEARS_FC constant
CapEx	= Depreciation (maintenance-only)	replace capex = dep in code
ΔNWC ratio	historical mean	ditto

5 · Realism-Upgrade Road-Map
Area	Practical Enhancement
Cost of Debt	Derive yield-to-maturity from traded corporate bonds or option-adjusted spreads instead of income-statement division.
Capital structure drift	Add user-defined target D/V and re-lever β each year.
ERP / rf	Pull up-to-date US 10Y for rf and use an implied ERP estimator (Damodaran / Barra) rather than a static 5 %.
Beta stability	Employ a 60-month exponential weighting or rolling window; optionally un-lever / re-lever against peer median.
Scenario WACC	Run Base / Bull / Bear WACCs to show sensitivity of intrinsic price.
Driver-rich FCF	Switch revenue CAGR to explicit Volume × ASP or SaaS ARPU × subscriber ladders; split CapEx into maintenance vs growth; model DSO / DIO / DPO individually.
Probabilistic valuation	Wrap the whole stack in scipy.stats.qmc.Sobol Monte-Carlo draws for growth, margin, WACC, and g; report percentile bands.
Refinancing calendar	Pull debt-maturity tables, forecast interest cost and amortisation rather than static Rd.
UI & persistence	Serve results via Streamlit, store daily outputs in DuckDB, trigger via GitHub-Actions CRON.

6 · File Layout
text
복사
.
├── dcf_intrinsic_value.py    # main driver (calls WACC, does DCF)
├── wacc_calc.py              # CAPM + statement-driven WACC
├── requirements.txt          # pip install -r
└── README.md                 # you are here
