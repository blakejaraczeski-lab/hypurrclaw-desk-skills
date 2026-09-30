---
name: funding-carry
description: Shadow tracker for Hyperliquid funding-rate carry, no trades. Use when Blake says "funding", "carry", "basis", "delta neutral", "funding scan", or "carry grade". Scans Hyperliquid perps for persistent funding, checks whether a spot leg exists to hedge, and logs a hypothetical hedged position so realized funding minus costs can be measured before any capital is used.
---

# funding-carry

A structural strategy, not a prediction: when perp funding is persistently positive, a long spot plus short perp of the same size collects funding with little price exposure. The questions are whether the funding persists, whether a hedge exists, and whether it pays after costs. This skill measures that in shadow. It never trades.

## 1. Scan (`funding scan`)
Keep it cheap. There is no single call that returns funding for every perp, so screen a fixed list instead of pulling history for all coins.
- One `searchTools` call for the ids of `hyperliquid market snapshot`, `hyperliquid funding history`, `hyperliquid spot markets`, and `hyperliquid orderbook`. Reuse them.
- **Hedgeable list only** (a Hyperliquid spot token exists for the same underlying): BTC perp with UBTC spot, ETH perp with UETH spot, SOL perp with USOL spot, HYPE perp with HYPE spot, PURR perp with PURR spot. Confirm each pairing once with `hyperliquid spot markets`; drop any that no longer lists.
- For each pair: current funding from `hyperliquid market snapshot`, then `hyperliquid funding history` for the last 7 days (this is at most 5 history calls).
- Funding at exactly 0.00125% per hour is Hyperliquid's baseline rate (about 10.95% APR), not a squeeze. That baseline is still carry: record it as such.
- Read both orderbooks: spread and depth within 0.5% at $50 notional per leg.
- Add one row for the highest current-funding perp outside the list, marked `hedge: none` (information only).

## 2. Shadow entry
For each eligible candidate, record a hypothetical $50 spot long plus $50 perp short at the current mids:
- `entry_cost` = both legs' spreads plus taker fees for entry **and** exit (round trip, both legs). Use fee values from the tool if shown; otherwise Unknown, never a guess.
- `apr_7d` = 7-day mean hourly funding x 24 x 365.
- `breakeven_days` = entry_cost / (daily funding on $50).
- Prediction: `claim` "net carry after costs is positive after 7 days," `p`, `invalidation` (funding negative for 24h straight, or spot/perp basis gap above 1%).

## 3. Save
Always stage the file, even when no row is eligible: a scan with zero eligible rows is still a data point. One `writeFile` to `/terminal/lab/carry-YYYYMMDD.md`:

```
# carry scan YYYY-MM-DD
made_at:

| coin | hedge | funding_pos_share_7d | apr_7d | spread_spot | spread_perp | entry_cost_usd | breakeven_days | p | invalidation | graded_at | funding_earned_usd | net_usd | result |
|------|-------|----------------------|--------|-------------|-------------|----------------|----------------|---|--------------|-----------|--------------------|---------|--------|
```

## 4. Grade (`carry grade`, 7 days after made_at)
- Read the file. For each row, sum the actual hourly funding since `made_at` on $50 (from `hyperliquid funding history`).
- `net_usd` = funding_earned_usd - entry_cost_usd. `result` HIT if net_usd is above 0, else MISS. INVALIDATED if the invalidation printed first.
- Rewrite the file with only the grading columns filled.

## 5. What counts
The sleeve is worth a rung step only after 20 or more graded rows with a positive median `net_usd` and no single row losing more than its entry cost. `lab` decides; Blake changes the rung. At $100 total, expect cents to low dollars per week; the point is proving the process, not the size.

## Reply shape (10 lines max)
Top candidates with apr_7d, hedge yes/no, breakeven_days, then `carry file staged, tap ✅`.

## Never
Open, preview, or stage positions. Treat a coin with no spot leg as hedged. Annualize a single hour of funding.
