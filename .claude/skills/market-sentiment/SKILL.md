---
name: market-sentiment
description: Read market mood (fear/greed, positioning, crowding) for stocks, ETFs, crypto, and commodities, and list what could push prices lower. Use when asked "what's the sentiment on X", "is the market fearful", "how crowded is X", "could X drop more". A mood reading, not a forecast.
---

# Market Sentiment

## Rules (read first)
1. Sentiment is a **mood reading, not a forecast**. Extreme fear can get worse. Extreme greed can last for months.
2. Use **at least 3 independent gauges**. One gauge alone = say "weak evidence".
3. Every gauge needs a **value, date, source link**. Missing = write "no data", never guess.
4. Don't turn a reading into a rule ("buy at fear < 20") unless it was backtested. If asked, backtest it (see below). Small samples (< 20 independent episodes) = say so.
5. No buy/sell advice. No invented probabilities.

## Gauges by asset class
| Class | Gauges (use 3+) |
|---|---|
| Stocks / ETFs | VIX level; CNN Fear & Greed; put/call ratio; AAII survey; % of stocks above 200-day average; price vs 52-week range |
| Crypto | Crypto Fear & Greed index; LunarCrush social data (`topic`, `cryptocurrencies`); funding rates and open interest (Crypto.com tickers / web); price vs 52-week range; CoinGecko `get-global-market` |
| Commodities | CFTC Commitments of Traders (speculator net long); ETF holdings/flows (GLD, SLV, GOLDBEES); gold/silver ratio; real yields and dollar trend; price vs 52-week range |
| Any | Social buzz (LunarCrush), volume spikes, news tone (use `market-news` skill) |

Price vs 52-week range = (price − low) / (high − low). Calculate it; show the inputs.

## Steps
1. Identify asset, class, and date. Ask if ambiguous.
2. Pull 3+ gauges from the table. WebSearch for ones without a connector.
3. Score each gauge: **Fear / Neutral / Greed**, with the thresholds you used.
4. Overall label only if gauges mostly agree. If they split, say "mixed".
5. **Downside-risk list** (max 5): what is crowded, what is stretched, which scheduled event could trigger selling. Each with evidence and source.
6. **Say what you did not check.**

## Output (keep it short)
```
ANSWER (1-2 lines): mood = <Fear|Neutral|Greed|Mixed>, evidence strength = <strong|weak>.

GAUGES:
| Gauge | Value | Date | Reads as | Source |

WHAT COULD PUSH LOWER (max 5):
- <item> | evidence | source

HISTORY CAVEAT: <e.g. "extreme fear in 2008 and 2020 was followed by higher prices 12 months later; 2 cases, not a rule">

NOT CHECKED: <list>
Not advice. Want the why?
```

## If asked to backtest a sentiment rule
Yahoo chart API works with explicit dates:
`https://query1.finance.yahoo.com/v8/finance/chart/<TICKER>?period1=<unix>&period2=<unix>&interval=1d` (use `-A "Mozilla/5.0"`).
Without `period1/period2` it returns sparse data. Compare forward 6- and 12-month returns after the signal against any random day. Count independent episodes (gap of 90+ days), report up/down counts, and state the weaknesses. Futures data is not ETF data; say so.

## Do not
- Quote an AI summary's numbers as verified.
- Call a bottom or a top.
- Mix spot and futures prices without saying so.
