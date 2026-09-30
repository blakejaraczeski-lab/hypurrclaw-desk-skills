---
name: universe-log
description: Daily base-rate dataset for Solana tokens. Use when Blake says "universe", "universe log", "base rate", "label yesterday", or "flush universe", run from chat each day. One screener call snapshots up to 25 eligible tokens (about 10 returned in testing) with their features, and a follow-up read labels the previous day's tokens 24h later (alive, dead, return). Produces the labeled rows the lab needs to test which gates actually predict survival.
---

# universe-log

The desk's biggest gap was data: zero graded rows. This skill produces 10 to 25 labeled rows a day for one or two taps, every token counted, not just the ones that pass. It is the backtest dataset.

## File
One file per UTC day: `/research/universe-YYYYMMDD.md`. It holds today's snapshot and the labels for **yesterday's** snapshot.

## 1. Label yesterday (if yesterday's file exists)
- `readFile` `/research/universe-<yesterday>.md`. Missing: skip this step and note `labels: none (no prior file)`.
- For each row in its `## snapshot`, one `market token snapshot` read (address, chain sol). At most 30 reads.
- Label each row:
  - `ret24` = mcap_now / mcap_then - 1 (2 decimals)
  - `liq_ratio` = liq_now / liq_then (2 decimals)
  - `dead`: yes if liq_now is under $2,000, or mcap fell more than 80%, or the read returns nothing tradeable
  - `alive`: yes if not dead and mcap_now is at least 0.5x mcap_then
  - A failed read is `unknown`, never dead or alive.

## 2. Snapshot today
- One `searchTools` call for the ids of `macro scan` and `analyze token`. Reuse them.
- One `macro scan` (v1) call: `mode market`, `chain sol`, `minLiquidityUsd 10000`, `minVolume24h 10000`, `minMarketCapUsd 30000`, `maxMarketCapUsd 10000000`, `limit 25`. Verified 2026-09-30: v1 caps `limit` at 25 and returned 10 rows with fields address, symbol, name, priceUsd, mcapUsd, liqUsd, vol24h (no top10, no age). `macro scan v2` in `mode token` requires a single address, and its `sortBy` must be one of trendingScore24, volume24h, liquidityUsd, marketCapUsd, priceChange24h, txnCount24h.
- The filters are deliberately looser than the A-setup gates so the file contains failures too. Record every returned token. Do not drop tokens that fail gates.
- **Copy each address exactly from the tool's `address` field.** Never retype a mint from memory; a retyped mint came back corrupted in testing. If unsure, write `unknown`.
- Optional, if the turn is short: `analyze token` on the 5 with the highest liquidity for authorities, tax, and top10. Others get `sec: n/a`.
- Flags per row: `g5` liq at least $15k, `g7` vol at least $25k and vol/liq at most 50, `g8` mcap $50k to $5M. `a_setup_screen: yes` only when g5, g7, g8 pass **and** `sec` is clean.

## 3. Output

```
# universe YYYY-MM-DD
snapshot_at:       # ISO UTC
source:            # chat
scan_filters:      # the macro scan params used
returned:          # number of tokens the screener returned

## snapshot
| address | symbol | mcap | liq | vol24 | g5 | g7 | g8 | sec | a_setup_screen |
|---------|--------|------|-----|-------|----|----|----|-----|----------------|

## labels (snapshot of <yesterday>)
| address | symbol | mcap_then | mcap_now | ret24 | liq_ratio | alive | dead |
|---------|--------|-----------|----------|-------|-----------|-------|------|

## summary
labeled: · alive: · dead: · unknown: · median_ret24: · a_setup_alive_rate: · others_alive_rate:
```

Full mints in every row. No prose outside the file.

## 4. Write
- One `writeFile` to `/research/universe-YYYYMMDD.md`. "Staged, tap ✅" until the receipt.
- **Chat only.** Verified 2026-09-30: cron runs do not store their full output (`no_outbox_rows`, 312-character preview), so a cron cannot produce this file and `flush` cannot land it.
- Long turns can die silently. If labels plus snapshot is too much for one turn, run `universe label` (step 1 only) and `universe snapshot` (step 2 only) as two messages; each stages its own write to the same day file (the second rewrites the file with both sections).

## 5. What the lab does with it
After 7 to 14 days (100+ labeled rows), `lab` can test claims like "a_setup_screen tokens stay alive more often than others" against the benchmark of all rows, out of sample on later days. `desk` reports the running alive and dead base rates.

## Budget
At most 38 tool calls: 1 read, up to 25 label reads, 1 search, 1 scan, up to 5 analyze.

## Never
Trade. Invent a label from a failed read. Filter the snapshot down to winners.
