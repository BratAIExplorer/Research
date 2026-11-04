---
name: "stock-research"
description: "Four stock-research modes for Bharat: EARNINGS (pre-earnings brief for one ticker), RISK (risk check of his own portfolio), SCREEN (shortlist from his criteria), COMPETE (sector/competitor landscape). Use when he says 'earnings brief', 'risk check my portfolio', 'screen for stocks', or 'compare competitors in [sector]'. For a plain 'research X' or 'review this stock', use stock-review-master instead."
---

# Stock Research (v1.1, 2026-11-04)

One skill, four modes. Personal investing only. Not for Fortress product work.

Style: blunt, short, readable by a 10-year-old. Define jargon once in brackets. Decision summary first, tables after.

## Shared rules (apply to every mode)

1. **No data = stop.** If a number cannot be found from a real source, write "NO DATA" in that cell. Never fill a gap with a plausible number. If a whole mode cannot run without it (no holdings, no criteria, no ticker), stop and ask.
2. **Every number carries a source and a date.** Label each as: reported / company guidance / analyst estimate / third-party / my calculation / my opinion. Say "I checked X" or "I'm assuming X", never blur them.
3. **Same non-result three times: stop and report it.**
4. **Conflicting sources: show both values and the dates.**
5. **Watch-only language.** Allowed labels: Wait, Watch, Speculative Watch, Avoid. Never "Buy". Never a dollar amount to invest. Never promise a return in a time window. Entry and stop-loss prices are NOT given; give "levels to watch" and "what would change my view". He sets his own loss limit and exit date.
6. **Read-only on brokerage.** IBKR tools may be used to READ data (prices, earnings insight, options, screener, positions). Never create or change orders, alerts, watchlists or saved screeners.
7. **Data sources, in order:** IBKR read tools if connected, then company filings / investor-relations page, then web. If a page is cut off or will not open, say exactly what was not read.
8. **Do not repeat what other skills do.** For a deep single-stock audit (pasted write-ups, one-offs, peer tables with FCF yield), hand off to `stock-review-master`. This skill points at it, it does not copy it.
9. **Disclaimer last line:** analysis, not financial advice.

## Mode picker

| He says | Mode |
|---|---|
| "earnings brief for X", "X reports soon" | EARNINGS |
| "risk check my portfolio", "what could hurt my holdings" | RISK |
| "screen for stocks that...", "find me stocks" | SCREEN |
| "who are the competitors in X", "compare the sector" | COMPETE |

If unclear, ask one short question. Do not guess the mode.

---

## Mode 1: EARNINGS

**Needs:** one ticker and exchange. If the ticker could mean two listings, stop and ask. Confirm the next earnings date from the company or IBKR; if none found, say so.

**Step 0 (always first): has this quarter already been reported?** Check the company's investor-relations page or latest 8-K for the most recent results date.
- If the quarter was already reported: say so in line one. Give a short RECAP instead (actual vs estimate, guidance, call takeaways, price reaction) and the next earnings date (or "NO DATA"). Do not write a preview of a quarter that is over. Lean stays Wait / Watch / Avoid.
- If it has not been reported yet: continue with the full brief below.
- If IBKR tools fail (e.g. "Service Unavailable"), say so, fall back to filings and web, and mark price and options-implied move "NO DATA" unless a dated source is found.

Output, in this order:
1. **Decision summary (top):** Lean = Wait / Watch / Avoid, one sentence why, what would change it. Not "Buy".
2. Next quarter expectations: revenue and EPS (analyst consensus, dated). Mark as estimate.
3. Last 4 quarters: actual vs estimate (beat / miss / in line) and the trend.
4. Key metrics analysts are tracking (e.g. margins, guidance, segment growth).
5. Last earnings call: 3 to 5 plain-English takeaways and any guidance change (old range vs new range). Read the release or transcript; if not read, say so.
6. Segment revenue and trends.
7. Price reaction after the last 4 reports (day-after move, from real price history).
8. Options-implied move (from option data, with the expiry used). No option data = "NO DATA".
9. Bull case and bear case with rough upside / downside as ranges, labelled opinion.
10. Check: one-offs flattering or hurting earnings, adjusted vs GAAP, acquisition-driven growth.

## Mode 2: RISK

**Needs:** his actual holdings (tickers + weights or amounts + account currency). Read from IBKR positions if connected and he agrees; otherwise ask him to paste. **No holdings = stop.**

Compute, do not guess. Pull daily price history and calculate with Python (use `python -I` on downloaded data). If history is missing for a holding, mark that holding "NO DATA" and exclude it from the stats, saying so.

Output:
1. **Decision summary (top):** the top 3 risks and the one change that cuts the most risk.
2. Correlation between holdings (matrix or the top 5 most-correlated pairs).
3. Sector, geography and currency exposure (% of portfolio).
4. Position sizing: any holding over ~20% or any tiny position that does not matter. Say the threshold is a rule of thumb, not a law.
5. Interest-rate sensitivity per position (qualitative unless duration/beta data exists).
6. Liquidity per asset (average daily volume vs his position).
7. Stress test: replay the portfolio through real past drawdowns (2008, 2020, 2022) using actual history where it exists. If a holding did not exist then, say so. Never present a beta-based guess as a replay.
8. Tail risks: name them, give a rough likelihood band (low / medium / high) labelled opinion. No fake precise percentages.
9. Hedging or reducing ideas, including one that simply removes something (sell or trim) and one unconventional idea. Described, not ordered.
10. Rebalancing: describe the direction of shifts (e.g. "trim X, add Y"), in % of portfolio, never dollars.
11. Heatmap-style table: holdings x risk type, with Low / Med / High.

## Mode 3: SCREEN

**Needs:** his criteria (market, sector, size, growth, dividend, risk appetite). **No criteria = stop and ask.** Prefer the IBKR screener (read-only preview/run, never create or change saved screeners) or a dated web screen.

Output:
1. **Decision summary (top):** how many names passed and the one-line pattern.
2. Up to 10 names that actually pass. If fewer pass, show fewer. Do not pad to 10.
3. Per name, one summary row: ticker, price and date, P/E vs industry, 5-year revenue growth, moat (weak / moderate / strong, opinion), dividend yield and whether it is covered by free cash flow, debt-to-equity (check units, leases), risk score 1 to 10 (opinion, one-line reason).
4. 12-month bull and bear price range per name, labelled opinion. Missing basis = "NO DATA".
5. Levels to watch (price areas) and what would change the view. No entry and stop-loss prices.
6. Label each: Watch / Speculative Watch / Avoid. Keep at most one quality, one value, one speculative name separate if he asks for top picks.
7. Say why obvious names were left out.
8. For anything he wants to act on, hand off to `stock-review-master`.

## Mode 4: COMPETE

**Needs:** a sector or a company (then find its peers). If the peer set is unclear, state the 5 to 7 you picked and why.

Output:
1. **Decision summary (top):** which company looks strongest in this group (labelled opinion) and the biggest risk to that view. Not "Buy".
2. Top 5 to 7 players: market cap with date.
3. SWOT for the top 2 only.
4. Market share trend, last 3 years. No reliable source = "NO DATA".
5. Revenue and margin comparison (table, same period for every company, say which period).
6. Moat per company: brand, cost, switching costs, network effects. Evidence, not adjectives.
7. Management: capital allocation track record (buybacks executed, M&A outcomes, dilution).
8. Innovation: R&D spend as % of revenue where reported.
9. Industry risks: regulatory, disruption, macro.
10. Catalysts in the next 12 months, with dates where known.

---

## Version history
v1.1 (2026-11-04): EARNINGS Step 0 added (check if the quarter was already reported; recap instead of preview) and IBKR-failure fallback. Found in a dry-run on AXTI.
v1.0 (2026-11-04): first version. Merges four prompt templates (earnings, risk, screener, competitive landscape) into one skill. Replaced "buy/sell" and entry/stop-loss outputs with watch-only language to match stock-review-master. Added shared no-data rule and read-only brokerage rule.
