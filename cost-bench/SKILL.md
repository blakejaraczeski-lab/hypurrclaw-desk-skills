---
name: cost-bench
description: Measures real trading costs with read-only quotes, no trades. Use when Blake says "cost bench", "slippage", "measure costs", "what would it cost to trade <token>", or when a run shows gate G9 costs UNKNOWN. Quotes buy and sell round trips at $5, $10, $25, $50, and $100 and saves a cost table by liquidity bucket that ca-intake uses for G9 and the lab uses for post-cost results.
---

# cost-bench

Edge smaller than costs is not edge. This skill builds the cost model the desk is missing, using quotes only. It never signs, stages, or executes a trade.

## 1. Pick mints
- Blake names a mint, or
- choose up to **3** mints from recent runs, one per liquidity bucket that has the fewest samples in the current file.

Buckets by pool liquidity: `L1` under $25k · `L2` $25k to $100k · `L3` $100k to $1M · `L4` over $1M.

## 2. Quote (read only)
- One `searchTools` call to find the Jupiter swap quote tool (and the Pump.fun token state tool for bonding-curve tokens). Run through `executeSafeReadTool` only. Never `executeWalletWriteTool`.
- For each mint and each notional in $5, $10, $25, $50, $100:
  - buy quote: USDC or SOL -> token
  - sell quote: the tokens from that buy -> back to the same quote asset
- Record per quote: in amount, out amount, price impact %, route, platform or route fee if shown, timestamp.
- Network fee: use the quote's fee estimate if provided; else record Unknown (don't guess).
- Round-trip cost % = 1 - (final quote-asset out / starting quote-asset in).
- Cap: 3 mints x 5 notionals x 2 sides = 30 quotes max per run.

## 3. File
Read the existing `/terminal/lab/costs-<chain>.md` if it exists, append the new samples, and recompute the summary.

```
# costs <chain>
updated_at:
method: read-only quotes (no fills); realized fills will replace quotes when micro-tests exist

## summary
| bucket | samples | median round_trip % at $5 | $10 | $25 | $50 | $100 |
|--------|---------|------|-----|-----|-----|------|

## samples
| utc | mint | ticker | liq_usd | bucket | notional | buy_impact % | sell_impact % | fees | round_trip % | route |
|-----|------|--------|---------|--------|----------|--------------|---------------|------|--------------|-------|

## caveats
- quotes are not fills; expect worse at execution
- bonding-curve and thin pools can move between quote and fill
```

One `writeFile`. "Staged, tap ✅" until the receipt.

## 4. How others use it
- `ca-intake` G9: look up the bucket and the intended notional (at rung OFF, use $10). PASS only if the median round trip is below half the expected move in the claim; else FAIL. Fewer than 3 samples in the bucket: UNKNOWN.
- `lab`: every result is reported net of the bucket's median round trip. Zero-cost results are invalid.

## Reply shape (8 lines max)
Buckets updated, median round trip at $10 per bucket, the worst quote seen, then `costs staged, tap ✅`.

## Never
Execute, sign, or stage a swap. Use chart prices as execution prices. Fill a missing fee with a guess.
