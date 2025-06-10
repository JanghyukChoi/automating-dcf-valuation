
This repository combines two self-contained, audit-friendly building blocks:

File	Role
wacc_calc.py	Computes a forward-looking WACC from market prices & latest statements.
dcf_intrinsic_value.py	Pulls the last 5Y financials, projects 5 Y FCF, discounts them with the WACC from wacc_calc.py, converts EV→equity value→intrinsic price, and benchmarks it to the spot market quote.

The result is a one-command valuation stack that updates straight from Yahoo Finance without manual inputs.

Code Flow in Detail

Step	Logic
Market data pull	5 years of daily closes for the stock and a supplied market proxy (default ^GSPC, KOSPI example 000660.KS).
Raw β	OLS of stock ΔP/P on market ΔP/P.
Blume adjusted β	β_adj = ⅔ β_raw + ⅓.
Re (CAPM)	rf + β_adj × ERP (defaults rf 3 %, ERP 5 %).
Rd	Latest interest expense ÷ total debt (auto-label search).
Tax rate	Income-tax-expense ÷ EBT.
Capital weights	Debt & market-cap from the same statement block.
WACC	E/V·Re + D/V·Rd·(1-Tc) returned as a dict with all sub-components.
