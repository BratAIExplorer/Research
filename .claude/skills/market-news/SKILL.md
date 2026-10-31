---
name: market-news
description: Get dated, source-checked news for a stock, ETF, crypto coin, or commodity (gold, silver, oil). Use when asked "what's the news on X", "why is X down/up", "any news risk for X". Reports facts and what is unverified. Gives no buy/sell advice.
---

# Market News

## Rules (read first)
1. Every price and headline gets a **date and a source link**. No date = say "undated, unverified".
2. Label each claim: **Verified** (2+ independent sources agree), **Single source**, or **Unverified** (AI summaries, forums, no link).
3. Sources disagree on a number? Show both. Never average them.
4. Can't find data for today? Say so and give the latest dated data. Do not fill gaps from memory.
5. News explains the past. It does not predict. Never write "will drop" or "will rise".
6. No buy/sell/hold advice. Say "not advice" once, at the end.

## Steps
1. **Identify the asset and class.** If the name could mean two things (e.g. a ticker on two exchanges), ask.
2. **Get the price first** (with timestamp):
   - Stock/ETF: IBKR `get_price_snapshot` if connected, else WebSearch (Kitco, Yahoo, Trading Economics).
   - Crypto: CoinGecko `get-coin-markets`, then `get-crypto-news`. Present headlines verbatim.
   - Commodity: WebSearch (Kitco, USAGOLD, Trading Economics). Futures and spot differ; say which.
3. **Search news** for the last 7 days, 2+ queries. Prefer: Reuters, Bloomberg, WSJ, Kitco, CoinDesk, company filings, central bank sites.
4. **Sort events into two bins:**
   - *Scheduled* (dated): Fed/RBI meetings, CPI/PCE, earnings, token unlocks, ETF decisions, OPEC.
   - *Unscheduled*: wars, hacks, downgrades, lawsuits, liquidations.
5. **Downside-risk list** (not a forecast): for each risk give *what*, *date if known*, *source*, *label*. Max 5 items, ranked by how soon.
6. **Say what you did not check.**

## Output (keep it short)
```
ANSWER (1-2 lines): what happened, what is the biggest near-term risk.

PRICE: <asset> <price> as of <date/time>, source <link>

WHAT HAPPENED (max 5):
- <fact> [Verified|Single source|Unverified] (<source>, <date>)

RISKS AHEAD (max 5):
- <risk> | <date or "undated"> | <label> | <source>

NOT CHECKED: <list>
Not advice. Want the why?
```

## Do not
- Trust a number just because an AI summary quotes it.
- Treat a headline as the cause. Write "reported cause", not "cause".
- Add adjacent assets nobody asked about.
