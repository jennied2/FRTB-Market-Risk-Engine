# FRTB Market Risk Engine

A self-contained, educational implementation of a market-risk capital engine built around the four pillars of the Basel **Fundamental Review of the Trading Book (FRTB)** Internal Models Approach (IMA). It measures market risk with **Value at Risk (VaR)** and **Expected Shortfall (ES)**, validates the model using the full regulatory **backtesting suite**, calibrates a **stressed-period** capital number, and runs a **P&L Attribution Test (PLAT)**.

The project deliberately uses a simple rolling-window historical-simulation model to expose a central insight of FRTB: a risk model can appear well‑calibrated on average, pass naive frequency tests, and earn a regulatory "green light", yet still fail the more sophisticated timing‑aware and tail‑sensitive tests — under‑protecting exactly when protection matters most.

---

## What the project does

The engine works with a three‑asset portfolio containing genuinely different risk behaviours: an equity ETF (**SPY**), a long‑Treasury ETF (**TLT**), and gold (**GLD**), weighted **50 / 30 / 20**. It pulls real market data via `yfinance` and automatically falls back to a synthetic, correlated, fat‑tailed return generator (with two embedded crisis windows) if the download fails or returns too little data.

The code is organised around four pillars:

1. **Measurement – VaR and ES engine**  
   One‑day‑ahead rolling VaR and ES via historical simulation (no distributional assumptions). VaR is the empirical quantile of the trailing loss window; ES is the mean of losses beyond that quantile. FRTB mandates ES at 97.5% confidence; VaR is also reported at 99%.

2. **Backtesting – proving the model is statistically accurate**  
   A complete validation suite:
   - **Basel Traffic Light** test (green / yellow / red zones), both full‑period and rolling 250‑day.
   - **Kupiec Proportion‑of‑Failures** test (unconditional coverage).
   - **Christoffersen Independence** test (detects clustering of exceptions).
   - **Christoffersen Conditional Coverage** (frequency + independence combined).
   - **Acerbi–Szekely** Z1 and Z2 statistics for backtesting Expected Shortfall, with Monte‑Carlo p‑values.
   - **Berkowitz** density‑forecast tail test.
   - **Bootstrap confidence intervals** for risk statistics such as ES.

3. **Stressed‑window calibration**  
   Slides a 250‑day window across the full history, selects the one that maximises ES, and reports the **stress multiplier** (stressed ES ÷ current ES). The worst window is labelled against a list of known historical crises (e.g. COVID‑19, the 2022 rate‑hike rout).

4. **P&L Attribution Test (PLAT)**  
   Compares the model’s theoretical (sensitivity‑based) P&L with the desk’s actual P&L using Spearman rank correlation and an unexplained‑variance ratio, then assigns a green / amber / red zone.

Finally, `main()` runs all four phases end to end, prints a formatted **daily market‑risk report**, and saves two diagnostic charts (`breaches.png` and `breach_intensity.png`).

---

## Requirements

- **Python 3.8+** (developed on 3.11)
- [`numpy`](https://numpy.org/) – numerical arrays and vectorised rolling‑window computations
- [`pandas`](https://pandas.pydata.org/) – time‑series handling
- [`scipy`](https://scipy.org/) – statistical distributions and tests (chi‑squared, Spearman, normal quantiles)
- [`yfinance`](https://pypi.org/project/yfinance/) – downloading real market data (optional at runtime; the engine falls back to synthetic data when unavailable)
- [`matplotlib`](https://matplotlib.org/) – diagnostic plots
- A Jupyter environment ([`jupyterlab`](https://jupyter.org/) or [`notebook`](https://pypi.org/project/notebook/)) to open and run the `.ipynb`

---

## Installation

Clone the repository and (recommended) create a virtual environment:

```bash
git clone https://github.com/jennied2/FRTB-Market-Risk-Engine.git
cd FRTB-Market-Risk-Engine
