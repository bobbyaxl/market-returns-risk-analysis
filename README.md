# The Anatomy of Market Returns

### Because a price chart is a terrible way to understand risk.

Markets have a funny habit of making simple things look complicated and complicated things look simple.

A chart goes up. Everyone feels smart.

A chart goes down. Everyone discovers the word *volatility*.

This project takes five very different assets, puts them on the same analytical playing field, and asks a less comfortable question:

> **What actually happens when we measure how they behave?**

Returns. Volatility. Correlation. Drawdowns. Tail risk.

The usual suspects.

---

## The Cast

Everything is expressed in **U.S. dollars**, with the return basis made explicit:

| Asset | Series | Return basis |
|---|---|---|
| **NIFTY 50** | `^NSEI`, converted using `INR=X` | Price index — dividends excluded |
| **S&P 500** | `^SP500TR` (with `^GSPC` fallback) | Total return where available |
| **Gold** | `GLD` | ETF, net of fees |
| **U.S. Treasuries** | `TLT` | Dividend-adjusted ETF return |
| **Bitcoin** | `BTC-USD` | Spot |

Two supporting series are used:

- `INR=X` — USD/INR, to put NIFTY into dollar terms
- `^IRX` — 13-week U.S. Treasury-bill yield, used as the risk-free rate

Different markets. Different personalities. Different ways of hurting you.

The point isn't to crown a winner.

The point is to make the comparison honest.

---

## The Question

**How different are assets when we measure their behaviour through returns, volatility, correlation, distributions, and drawdowns rather than simply comparing price charts?**

Because "this went up more" isn't exactly quantitative finance.

It's a bar conversation.

---

## Six Questions

The notebook answers six questions from the computed results:

1. **Which asset compounded the most, and which the least?**
2. **Did higher volatility come with higher returns?**
3. **Which assets suffered the deepest drawdowns?**
4. **Which assets moved least with the others, and is that stable over time?**
5. **How heavy are the tails, and are returns consistent with a normal distribution?**
6. **Does the ranking of assets depend on the performance measure used?**

The Findings section generates the answers from the tables above it, so the prose doesn't quietly drift away from the numbers.

---

## What We're Doing

The analysis covers:

- historical prices
- simple returns
- CAGR
- arithmetic annualised return
- volatility
- rolling volatility
- daily and weekly correlation
- lead-lag correlation
- year-by-year correlation
- skewness
- excess kurtosis
- Jarque-Bera normality testing
- maximum drawdown
- time under water
- recovery time
- worst single-observation losses
- historical VaR
- Expected Shortfall
- downside deviation
- Sharpe ratio
- Calmar ratio
- currency robustness

In other words, we're trying to find out what happens **after the chart stops being pretty**.

---

## The Mathematics

Simple returns:

$$
r_t = \frac{P_t}{P_{t-1}} - 1
$$

CAGR:

$$
\text{CAGR} = W_T^{\,1/\text{years}} - 1,
\qquad
W_T = \prod_{t=1}^{T}(1+r_t)
$$

Annualised volatility:

$$
\sigma_{\text{annual}} =
\sigma_{\text{obs}}\sqrt{N}
$$

where $N$ is the observed number of return observations per year in the common sample.

Drawdown:

$$
DD_t =
\frac{W_t}{\max(W_0,\ldots,W_t)} - 1,
\qquad W_0=1
$$

Sharpe ratio:

$$
\text{Sharpe} =
\frac{\bar r_{\text{excess}}\,N}
{\sigma_{\text{excess}}\sqrt{N}}
$$

The T-bill version uses the volatility of **excess returns**, not raw returns.

Historical VaR is the empirical $(1-c)$ quantile of returns. Expected Shortfall is the average return beyond that threshold.

No crystal balls.

No "AI-powered alpha engine."

Just mathematics, data, and the occasional uncomfortable result.

---

## Why the Comparison Needs Some Discipline

Markets don't all behave on the same clock.

India closes before the U.S. opens. Bitcoin trades continuously. FX has its own timestamp conventions.

A naive daily correlation can therefore make two markets look less related than they actually are.

So the notebook shows:

- daily correlation
- weekly correlation
- a lead-lag check
- year-by-year weekly correlations

Weekly returns reduce the timing problem. They don't magically make it disappear.

The project also converts NIFTY into USD before comparing it with dollar-denominated assets. A separate INR-versus-USD robustness check shows how much that choice matters.

---

## Reproducibility

This is deliberately designed to run in **Google Colab** without a local Python installation.

The sample end date is fixed at:

`2026-09-30`

On the first run:

1. the notebook downloads the public Yahoo Finance data;
2. it writes `data/market_data.csv`;
3. it writes `data/snapshot_info.json`;
4. those files should be committed to the repository;
5. subsequent runs use the frozen snapshot and do not contact Yahoo.

That means the results don't quietly change because the underlying data changed six weeks later.

### Google Colab

Colab starts in `/content`, not in the repository.

Clone the repository and move into it before running:

```python
!git clone <repo-url>
%cd market-returns-risk-analysis
```

Then, if necessary:

```python
%pip install -q -r requirements.txt
```

After the first successful run, download and commit:

```text
data/market_data.csv
data/snapshot_info.json
```

The notebook can then be rerun offline from the committed snapshot.

---

## Repository Structure

```text
market-returns-risk-analysis/
│
├── README.md
├── market_returns_risk_analysis.ipynb
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── README.md
│   ├── market_data.csv          # created by first live run
│   └── snapshot_info.json       # created by first live run
│
└── src/
    └── README.md
```

The first version intentionally keeps the analytical workflow visible inside the notebook.

As the projects become more sophisticated, reusable functions can move into `src/`.

---

## Data Quality

Before the common-date sample is constructed, the notebook checks the raw data for:

- missing observations
- duplicate timestamps
- non-positive prices
- missing supporting series
- implausible single-period moves
- suspiciously long flat-price runs

These checks are deliberately performed **before** rows are dropped.

Otherwise you can make missing data disappear and then congratulate yourself for finding no missing data.

That would be a neat trick.

It would also be wrong.

---

## Limitations

There are plenty.

1. **NIFTY is still a price index.** Dividends are excluded, so its return is understated relative to a total-return NIFTY series by roughly its dividend yield, typically around 1% a year.
2. **Weekly returns reduce but don't eliminate non-synchronicity.**
3. **Return intervals aren't perfectly equal.** Holidays and the common-date join mean some observations span several calendar days.
4. **Bitcoin weekends are dropped by the common-date join.** Weekend moves therefore enter through the following common observation.
5. **GLD and TLT are ETF proxies**, not pure asset-class indices.
6. **Bitcoin has a relatively short history** compared with traditional assets.
7. **Year-by-year correlations are noisy.** A year's estimate may contain only about 52 weekly observations and no confidence intervals are reported.
8. **Historical VaR and ES are only as good as the observed tail.**
9. **Yahoo Finance is an unofficial data source** and values can occasionally be revised or wrong.
10. **FX timestamps differ slightly from equity timestamps.**
11. No transaction costs, taxes, liquidity constraints, portfolio weights, or confidence intervals are modelled.
12. This is a **descriptive analysis**, not a trading strategy or investment recommendation.

Knowing what the model can't tell you is at least as important as knowing what it can.

---

## What Changed from Version 1

| Problem | Fix |
|---|---|
| Ambiguous annualised return | CAGR and arithmetic annualised mean are separated |
| NIFTY in INR vs dollar assets | NIFTY converted to USD |
| Price vs total-return mismatch | Return basis explicitly documented |
| Hard-coded 252 | Observed sample frequency |
| Same-day cross-market correlation bias | Weekly correlation + lead-lag analysis |
| Floating sample window | Fixed end date |
| No reproducible data state | Frozen CSV + snapshot metadata |
| Data checks after dropping rows | Checks run on raw data |
| Zero-risk-free-rate Sharpe only | T-bill Sharpe added |
| Incorrect drawdown start | Wealth begins at 1.0 |
| No tail-risk depth | Expected Shortfall added |
| No downside measure | Downside deviation added |
| No recovery analysis | Recovery days and open/closed underwater spells |
| Findings were just questions | Findings generated from computed results |
| Broken equation rendering | `$$ ... $$` blocks |
| Hidden warnings | Warnings are surfaced |
| Weekly empty periods | Empty weeks dropped before `pct_change` |

---

## What's Next

This project is the foundation.

The next questions are more interesting:

- What happens when we combine these assets?
- Can we construct a portfolio that improves the risk/return trade-off?
- How stable are correlations across market regimes?
- What happens when we introduce portfolio weights?
- Can bootstrap methods tell us how uncertain these estimates are?
- Can factors explain the differences in return and risk?
- Can we build a backtesting framework without fooling ourselves?

Those become the next projects.

For now, we're starting with something simpler:

**Understand the damn data.**

---

## Final Takeaway

A market isn't a number.

Neither is risk.

And a return without context is just a number wearing a nice suit.

The framework is:

**Price → Return → Distribution → Risk → Correlation → Drawdown → Risk-adjusted performance**

The discipline is making sure the things being compared are actually comparable.

Everything else comes later.
