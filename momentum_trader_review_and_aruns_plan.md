# momentum_trader: Review and Plan for TradingBots-Aruns Project

Repo reviewed: https://github.com/luram94/momentum_trader (MIT license, last commit seen 2026-10-27)
Reviewed: 2026-10-30

## 0. Bottom line

- It is a decent **research scanner**. It is not a trading bot. There is zero broker or order code in it (I grepped for ibkr/alpaca/broker/order: nothing).
- The idea behind it (rank stocks by multi-period momentum, only buy when the market is healthy, rebalance weekly) is far more testable for a solo retail trader than the spike-chasing bot we discussed earlier. It uses daily data, so speed does not matter.
- **Do not trust its backtest numbers yet.** The backtest has a survivorship/selection bias that will make results look better than reality (section 3, problem 1).
- Recommendation: **port the logic, not the app.** Fix the backtest, shadow-trade it, then paper trade on IBKR. No real money until the gates in section 6 pass.

## 1. What I checked and what I did not

Checked (read the code myself):
- `README.md`, `config.yaml`, `requirements.txt`, `.github/workflows/snapshot.yml`
- `hqm/database.py` (universe screen, returns, scan logic)
- `hqm/backtest.py` (universe, scoring, rebalance, stops, simulation loop)
- `hqm/market_regime.py` (regime rules)
- File list and line counts (about 8,300 lines in total, including tests)

Did NOT do:
- I did not run the app, the tests, or a backtest. Any claim about its performance is therefore unknown.
- I did not read `risk_metrics.py`, `snapshot.py`, the Streamlit pages, or the tests line by line.
- I have **not seen the TradingBots-Aruns Project folder**. Section 5 is written without knowing its structure. Verify against the real folder before acting.

## 2. What it does

1. Pulls every NYSE/NASDAQ stock with market cap of $2B or more from Yahoo Finance (via `yfinance`).
2. Computes 1M, 3M, 6M and 1Y returns.
3. Ranks each stock 0-100 per period. Drops any stock below the 25th percentile in ANY period.
4. HQM score = average of the four percentiles. Top N, equal weight.
5. Market regime from QQQ vs its 10/20/50/200 day averages: UPTREND (full size), CAUTION (50%), DOWNTREND (0%, cash).
6. Backtest: weekly rebalance, 0.1% slippage each side, 0 commission, optional stop at the entry-day low capped by ATR, partial exit after N days, trailing stop on a moving average.

## 3. Problems (most important first)

**1. Survivorship and selection bias in the backtest. This is the big one.**
- `get_backtest_universe()` (`hqm/backtest.py:1122`) reads the `stocks` table. That table is *today's* list of $2B+ companies, ordered by *today's* market cap, then capped at 200 (`max_tickers`, `hqm/backtest.py:52`).
- So the backtest only trades today's 200 biggest companies. Companies that were big then and later collapsed or got delisted are missing. Companies that grew huge since are included. Both make a momentum strategy look better than it was in real time.
- Fix needs point-in-time data: who was in the universe on each past date, including delisted names. Yahoo does not give this reliably. I do not know what a proper dataset costs; that needs checking.

**2. Backtest ranks a different universe than the live scanner.**
- Live scan ranks against the full $2B+ list. Backtest ranks among 200 names. Percentiles and the "25th percentile" filter mean different things in each. The code docstring says the backtest "mirrors the live scanner". With a 200-name cap, it does not fully.

**3. Same-day signal and fill.**
- Scores use the close of day D, and trades fill at the close of day D (`_execute_rebalance`, `hqm/backtest.py:566`). You cannot know the close before the close. Realistic version: signal at close, fill at next open (or use closing-auction orders deliberately).

**4. Costs are light.**
- 0.1% slippage is plausible for large caps. Commission is 0. IBKR charges commissions; check your plan. Small effect on weekly rebalancing of 8 names, but it should be modeled.

**5. Data source risk.**
- `yfinance` is an unofficial scraper of Yahoo. It breaks without notice (the repo itself has workarounds for crashes, e.g. `threads=False`). Fine for research, **not** for anything that sends real orders. For execution use IBKR data.

**6. No benchmark comparison visible.**
- I did not see the backtest compare against simply holding QQQ or SPY. Without that, a "profit" means nothing. Add it.

**7. Complexity.**
- Stops, partial exits, trailing MA, regime scaling are all tunable knobs. More knobs means more ways to overfit a short backtest. Default config backtest is only 90 days, which proves nothing.

## 4. What is good

- Regime rules are simple, readable, and unit tested (`tests/test_market_regime.py`).
- Backtest was visibly fixed for earlier bugs (the docstrings mention removing fabricated returns and one-sided rebalancing). Someone cared.
- Real test suite and CI exist.
- MIT license: we can reuse code if we keep the license notice.
- Config is externalized (`config.yaml`).

## 5. How it fits TradingBots-Aruns Project

Rules for this work (from your standing rules):
- That folder runs **real bots**. Do not edit live bot files. Put everything in a **new, separate subfolder or branch**.
- Before touching anything: confirm which folder is the real one. Two similar folders means stop and ask you.
- Document every change.

What to take, what to leave:

| Take | Leave |
|---|---|
| Scoring logic (percentile ranks, HQM score, quality filter) | Streamlit UI, all `pages/` |
| `hqm/market_regime.py` (classifier + exposure scaling) | SQLite watchlist and portfolio tables |
| `hqm/backtest.py` as a starting point, after fixes | Nightly snapshot GitHub Action |
| `tests/test_market_regime.py` as a pattern | Docker, devcontainer, theme files |

Two cheaper options before porting anything:
- **Remove something:** skip the stops, partial exits and trailing logic. Test plain "top N by HQM, weekly, regime on/off" against "hold QQQ". If the simple version does not beat QQQ, the extras will not save it.
- **Unconventional:** use only the regime classifier as an on/off risk switch for your *existing* bots, and measure whether it would have reduced their drawdowns. This is the smallest change that might help real money.

## 6. Phased plan with gates

Gate numbers below are my proposed defaults. You decide them.

**Phase 0: setup (read-only)**
- Identify the correct Aruns folder. Create a new subfolder (suggest `momentum_research/`). Do not modify existing bot code.
- Copy in only the "Take" files. Keep the MIT notice and credit luram94.

**Phase 1: make the backtest honest**
- Fill at next open, not same close.
- Rank against the full universe, not 200 names.
- Point-in-time universe including delisted names (find a data source; cost unknown).
- Add buy-and-hold QQQ and SPY comparison.
- Add IBKR commissions.
- Test across at least one bad period (2022) and a good one, not 90 days.
- Gate: after costs, beats QQQ buy-and-hold on return-per-drawdown over the full test, and holds up if you shift the rebalance day. If not, stop here. Cheap to find out.

**Phase 2: shadow mode (no orders)**
- Script runs weekly and writes target weights to a CSV plus a log. No broker connection.
- Run 8-12 weeks. Compare what it said to buy with what actually happened.
- Gate: shadow results roughly match the fixed backtest.

**Phase 3: IBKR paper trading**
- Use `ib_async` (the maintained successor of `ib_insync`; confirm current name before installing).
- Hard guards in code: connect only to the paper port; refuse to start unless the account ID starts with `DU`; a single kill-switch file that stops all orders; max position size cap; limit orders only.
- Reconcile positions against the target after every rebalance and log mismatches.
- Note: paper fills on liquid large caps are reasonably realistic; they are not for thin stocks. Real-time data on paper needs a market-data subscription.
- Gate: 8-12 weeks, paper results track shadow results, zero unexplained order errors.

**Phase 4: live decision**
- Yours alone. If yes, start tiny, and only after Phases 1-3 gates pass.

## 7. Open questions for you

1. Which exact folder is TradingBots-Aruns Project (path)? Is there a similarly named copy?
2. Do the existing Aruns bots already talk to IBKR? If so, which library, and can this reuse that connection code?
3. Budget for a point-in-time data source? Or accept Yahoo-only results with a loud "biased" label?
4. What is the real capital you would eventually trade? (Matters for position sizing and whether commissions matter.)

## 8. Paste-in prompt for the Claude session inside TradingBots-Aruns Project

```
Read momentum_trader_review_and_aruns_plan.md first.
Rules: do NOT edit any existing bot file. Work only in a new subfolder
momentum_research/. Paper trading only, never live. Document every change.
First task (Phase 0 only): list the folder structure, tell me which files look
like live bots, confirm the path with me, then stop and wait.
```
