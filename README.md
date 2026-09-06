# Fixed-Income FRN Pricing, CVA and Hedging

Pricing, credit valuation adjustment, and interest-rate hedging of a
EUR-denominated capped/floored floating-rate note (UniCredit Variable Rate
Bond 2034, ISIN IT0005599110), built from a bootstrapped EUR swap curve.

## Structure

Ordered the way the analysis is actually built: data first, then pricing,
then risk.

- `helpers/` — shared modules: business-day calendar and day counts
  (`fi_calendar.py`), discount-curve I/O and shifting (`fi_curve.py`),
  bond cash-flow and option pricing (`fi_bond.py`), CVA and credit DV01
  (`fi_credit.py`).
- `data/` — market data and generated intermediate outputs (EURIBOR/IRS
  quotes, TARGET holidays, the vol surface, CDS spreads, PCA factors).
- `00_product_overview/` — the note's term sheet and cash-flow structure.
- `01_data_curve_construction/` — EUR curve bootstrapping and interpolation.
- `02_pricing/` — floater + cap/floor decomposition, Displaced Black
  caplet/floorlet pricing, risky (CVA-adjusted) valuation.
- `03_risk_sensitivities_hedging/` — PCA on the swap curve, key-rate DV01,
  hedge construction, credit DV01.
- `04_portfolio_risk/` — factor-model VaR/ES, Monte Carlo cross-check, risk
  decomposition.

## Methodology summary

- Curve: EUR swap curve bootstrapped from EURIBOR/IRS quotes, Modified
  Following business-day convention, ACT/360 (floating) / 30/360 (fixed)
  / ACT/365 (option time) day counts.
- Note: floater + long floor − short cap, capped/floored coupon priced via
  the Displaced (shifted-lognormal) Black model; a single flat vol per
  cap/floor strip is used rather than full caplet stripping — a disclosed
  simplification.
- Credit: reduced-form flat-hazard survival probability, CVA from
  notional-loss-on-default (dominant term); credit DV01 by central finite
  difference on the CDS spread.
- Risk: PCA level/slope/curvature factors on the swap curve, key-rate
  duration hedging, linear factor-model VaR/ES with Euler risk
  decomposition, cross-checked against Monte Carlo.

Notebooks import the `helpers/` modules directly; run them in the folder
order above (`01` → `02` → `03` → `04`) to reproduce all generated data
under `data/`.
