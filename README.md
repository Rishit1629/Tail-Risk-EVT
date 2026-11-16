# Tail Risk Modelling of the S&P 500 with Extreme Value Theory

Estimating the probability and size of extreme daily losses in SPY (2000–2025)
using the Peaks-Over-Threshold (POT) method and a Generalized Pareto
Distribution (GPD), and comparing it against Gaussian and historical VaR.

## Why this project
Normal-distribution risk models underestimate crashes because real market
returns have heavy tails. Extreme Value Theory models the tail directly
instead of assuming it looks like the middle of the distribution.

## Data
- SPY daily adjusted closes via `yfinance`, 4 Jan 2000 – 31 Dec 2025
- 6,538 daily observations; losses are defined as negative log returns

## Notebooks
| Notebook | What it does |
|---|---|
| `Tail Risk Model 01.ipynb` | Fits and validates the EVT model, computes VaR, backtests, and checks against historical crashes |
| `Tail Risk Model 02.ipynb` | Uses the fitted parameters for Monte Carlo simulation, crash probabilities, expected event frequency and risk-budget sizing |

## Method

**Notebook 01**
1. Compare a Gaussian tail threshold with empirical quantiles of daily losses.
2. Choose the POT threshold using a mean-excess plot and by fitting the GPD at
   the 95%, 97.5%, 99% and 99.5% quantiles (shape-parameter stability).
3. Quantify parameter uncertainty with bootstrap confidence intervals.
4. Check the fit with Q-Q and P-P plots, a Kolmogorov-Smirnov test and a
   parametric-bootstrap Anderson-Darling test.
5. Compute EVT VaR and compare with Gaussian and historical VaR:
   VaR_p = u + (β/ξ) · [ ((n/N_u)(1−p))^(−ξ) − 1 ]
6. Backtest out-of-sample (80/20 chronological split) and compare worst
   daily losses in four historical crashes against model VaR.

**Notebook 02**
1. Simulate 100,000 tail losses from the fitted GPD.
2. Convert to unconditional probabilities of daily losses exceeding fixed levels.
3. Translate probabilities into expected waiting times between events.
4. Compute Expected Shortfall and size a position from a VaR-based risk budget.

## Fitted model (97.5% threshold)
| Parameter | Estimate | 95% bootstrap CI |
|---|---|---|
| Threshold u | 2.52% daily loss (164 exceedances) | – |
| Shape ξ | 0.243 | [0.03, 0.41] |
| Scale β | 0.0095 | [0.0076, 0.0121] |

ξ > 0 indicates a heavy (power-law) tail. Goodness of fit: KS test p = 0.88,
Anderson-Darling bootstrap p = 0.56, so there is no evidence against the GPD.

## Results

**Value-at-Risk (daily loss, full sample)**
| Confidence | Gaussian | Historical | EVT |
|---|---|---|---|
| 99% | 2.81% | 3.47% | 3.50% |
| 99.5% | 3.12% | 4.49% | 4.39% |
| 99.9% | 3.75% | 7.46% | 7.16% |
| 99.99% | 4.51% | 10.79% | 13.57% |

At 99.99%, the Gaussian VaR is about a third of the EVT estimate. Three of the
four crash windows tested (dot-com, 2008, COVID) had a worst day above the
Gaussian 99.99% VaR; none exceeded the EVT figure.

**Out-of-sample breach rates** (VaR fitted on first 80% of data, tested on the
last 1,308 days)
| Confidence | Expected | Gaussian | Historical | EVT |
|---|---|---|---|---|
| 99% | 1.00% | 1.30% | 0.61% | 0.54% |
| 99.5% | 0.50% | 0.92% | 0.15% | 0.15% |
| 99.9% | 0.10% | 0.46% | 0.00% | 0.00% |
| 99.99% | 0.01% | 0.15% | 0.00% | 0.00% |

The Gaussian model breaches more often than expected at every level, while EVT
and historical VaR are conservative.

**Estimated frequency of large daily losses (Monte Carlo)**
| Daily loss exceeds | Probability per day | Roughly once every |
|---|---|---|
| 5% | 0.33% | ~1.2 years |
| 10% | 0.030% | ~13 years |
| 15% | 0.0067% | ~59 years |
| 20% | 0.0021% | ~189 years |

## Limitations
- GPD is fitted to exceedances assumed to be independent and identically
  distributed; volatility clustering is not modelled (a GARCH-EVT model would
  address this).
- Results depend on the threshold choice; shape estimates vary across
  thresholds and their confidence intervals widen sharply above the 99% quantile.
- The out-of-sample test has about 1,300 days, so at 99.9% and 99.99% only
  0–1 breaches are expected and the backtest cannot statistically separate the
  models there. No formal Kupiec/Christoffersen test is applied.
- The test period starts around late 2020, so the March 2020 crash falls in
  the training sample.
- Crash validation uses full-sample VaR, so it is in-sample.
- Notebook 02 uses parameters hard-coded from notebook 01.

## Possible extensions
- GARCH-filtered EVT for time-varying volatility
- Formal VaR backtests (Kupiec, Christoffersen)
- Multi-asset or portfolio tail dependence

## How to run
pip install -r requirements.txt
Run `Tail_Risk_Model_01.ipynb` first, then `Tail_Risk_Model_02.ipynb`
(in Jupyter or Google Colab).

## Tech
Python, NumPy, pandas, SciPy, matplotlib, yfinance
