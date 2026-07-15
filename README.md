# APEX — Build Journal

A public log of building **APEX**, a quantitative momentum stock-selection system, from scratch as a solo project. The code is private; this journal tracks the progress, the results, and — just as importantly — everything that *didn't* work.

**What it is:** every 30 days, APEX picks 3–10 S&P 500 stocks using price-based momentum signals, with a market-breadth regime filter that scales position count down (all the way to cash) as market health deteriorates. Positions are sized proportionally to signal strength.

**The philosophy:** most public backtests are quietly broken — survivorship bias, look-ahead leaks, in-sample weight fitting, cherry-picked start dates. This project's rule is that every claimed result must survive an honest attempt to destroy it. The failures below are documented with the same care as the wins.

---

## Current state (2026-07-14)

| Metric | Value |
|---|---|
| Clean-window backtest (2023-06 → present, 37 periods) | **+259% net** vs SPY +38.6% |
| Full-window backtest (2020-03 → present, 67 periods) | +586% net, ann. Sharpe 1.02 |
| Monte Carlo significance (clean window, 10k sims, survivorship-corrected) | **p = 0.0056** |
| Walk-forward validation (zero look-ahead, clean window) | **p = 0.016** — significant |
| Walk-forward validation (full window) | p = 0.48 — **not** significant |
| Deflated Sharpe Ratio (assuming 50 configs tried) | 0.74–0.92 — below the 0.95 bar |
| True daily-resolution max drawdown | **−43.9%** |
| Average weight of the single largest position | 44% (peak: 94%) |

The honest summary of those numbers: the system has a statistically significant edge **conditional on the post-2022 momentum regime**, it cannot yet be distinguished from selection luck at 95% confidence given the sample size, and living through it means enduring drawdowns that period-level tables hide. Paper trading runs continuously; a small live allocation with a hard stop-loss rule is the next phase.

---

## Timeline

**2026-07-14** — Started this public journal. Fixed a daily-automation bug where the price refresh skipped nearly all tickers every other day.

**2026-07-04 — Validation hardening.** Prompted by an honest audit ("will it actually make money?"):
- Found and fixed a survivorship bug in the validation script itself — the flagship p-values had been computed on a survivorship-biased universe. Significance *survived* the fix (p=0.0056).
- Added **walk-forward validation**: every period is scored with signal weights estimated only from *prior* periods. Result: the 2023+ window stays significant (p=0.016), but the full 2020+ window does not (p=0.48). Conclusion: **the edge is regime-conditional**, not universal. A trader starting in 2020 with honest expanding-window calibration would not have found it until 2023.
- Added the **Deflated Sharpe Ratio** (Bailey & López de Prado): with ~50 configurations tried during development and only 34–66 periods of data, the observed Sharpe can't yet be distinguished from the best of 50 lucky configs. Only more live periods fix this.
- Anchor-date sweep: all 6 rebalance-day shifts profitable — the edge isn't a calendar artifact.
- Daily-resolution stress report: true compounded equity drawdown is −43.9%, far worse than period tables suggest; the top pick averages 44% of the portfolio. These are the real expectations.

**2026-07-02 — Live trading plan adopted.** Order-sheet workflow (system generates, human reviews and executes — nothing auto-trades), a −25% portfolio hard stop validated to have *zero* historical cost, and written operating rules. Paper portfolio continues in parallel as the control.

**2026-06-24 — Execution & universe experiments: all dropped.** Hysteresis exit bands, adding S&P 400 midcaps, and multi-day staggered entry all traded cumulative return for marginally smoother Sharpe. A recurring theme by now: momentum's edge concentrates in large caps and in staying fully exposed.

**2026-06-12 — Audit day.** The paper portfolio's headline (+437%) looked too good, so I audited it: 27 of 28 closed periods turned out to be backfilled replay at historical prices, not live tracking — only 1 period was genuine forward performance. The display now splits the two honestly. Also found and fixed a price-cache staleness bug that had live picks computed on week-old prices. Automated daily syncs via Task Scheduler.

**2026-05-30 — Institutional techniques: all dropped.** Regime-adaptive signal weights (overfits), dollar-neutral long/short (shorting weak S&P names bleeds in a bull market: +447% → +47%), and factor orthogonalization (removes exactly the shared momentum component that *is* the alpha: +447% → +44%). Core finding: **APEX's edge is a concentrated bet on the momentum factor — techniques that diversify, hedge, or decorrelate destroy it.**

**2026-05-24 — Survivorship bias corrected.** Integrated historical S&P 500 membership (per-period universe filter + price histories of ~94 removed members). Impact was brutal: the base variant's cumulative return fell from +247% to +82%. The 2020–2022 numbers had been heavily inflated. Every result since is survivorship-corrected.

**2026-05-15 / 05-16 — Parameter sweeps.** Position caps, pick counts, breadth thresholds, score exponents, volatility targeting, and three academic signals (idiosyncratic vol, short-term reversal, residual momentum). Vol targeting's lesson generalized: momentum's best periods are its highest-vol periods, so anything that trims volatility trims exactly the return. Settled on uncapped score-proportional sizing with 10/6/3 picks per regime.

**2026-05-01 — Debiased validation.** Out-of-sample test with signal weights derived only from in-sample data: still significant (p=0.018). Added drawdown email alerts and a position-cap variant.

**2026-04-28 — Signal recalibration.** Re-measured all signal ICs; dropped one signal (6-month Sharpe) for negative predictive power. First full Monte Carlo validation: p=0.007 vs 10,000 random portfolios.

**Early 2026 — Foundations.** Started from pure 12-month momentum (measured rank IC +0.12), grew to a 17-variant ablation framework over 7 price signals, of which only 4 earned a place in the composite. Built the data layer: price caches immune to cloud-sync corruption, fundamentals from Polygon, historical constituent handling.

---

## Lessons so far

1. **Simplicity won every fight.** Seventeen variants, three institutional techniques, five academic signals, and a dozen execution tweaks were tested. Almost everything lost to a 4-signal weighted composite with proportional sizing.
2. **Your backtest is lying to you until proven otherwise.** Survivorship bias alone was worth −165pp. Stale prices, look-ahead entries, and in-sample weight contamination each independently inflated results before being caught.
3. **Sharpe and return are different goals.** Every risk-reduction overlay (vol targeting, caps, tighter stops, staggered entry) improved Sharpe and cost cumulative return. Pick your objective *before* the sweep.
4. **The audit instinct pays.** The best single day of this project was distrusting my own +437% headline.
5. **Significance has an expiry date on its assumptions.** A p-value computed on regime-conditional data is a claim about that regime, not the future.

---

## Disclaimer

This is a personal research project, published for transparency and learning. Nothing here is investment advice. Backtested results — even carefully debiased ones — do not predict future returns; the walk-forward section above shows exactly how an honest-looking edge can be regime-dependent. Do not trade money you cannot afford to lose based on anything you read here.
