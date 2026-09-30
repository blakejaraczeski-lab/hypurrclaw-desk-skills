---
name: universe-log
description: Daily base-rate dataset for Solana tokens. Use when Blake says "universe", "universe log", "base rate", "label yesterday", or "flush universe", and in the Universe Log cron. One screener call snapshots up to 30 eligible tokens with their features, and a follow-up read labels the previous day's tokens 24h later (alive, dead, return). Produces the labeled rows the lab needs to test which gates actually predict survival.
---

# universe-log

The desk's biggest gap was data: zero graded rows. This skill produces 30 labeled rows a day for one tap, every token counted, not just the ones that pass. It is the backtest dataset.

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
- One `searchTools` call for the ids of `macro scan v2` and `analyze token`. Reuse them.
- One `macro scan v2` call: `mode token`, `chain sol`, `minLiquidityUsd 10000`, `minVolume24h 10000`, `minMarketCapUsd 30000`, `maxMarketCapUsd 10000000`, `launchpadStatus migrated`, `includeScams false`, `limit 30`, `sortBy volume`. The filters are deliberately looser than the A-setup gates so the file contains failures too.
- Record every returned token (up to 30) with the screener's own fields. Do not drop tokens that fail gates.
- `analyze token` on the 10 with the highest liquidity only, for authorities and tax. The other 20 get `sec: n/a`.
- Mark screener gates per row from the fields you have: `g5` liq at least $15k, `g6` top10 at most 35%, `g7` vol at least $25k and vol/liq at most 50, `g8` mcap $50k to $5M. `a_setup_screen: yes` when all four pass and `sec` is clean.

## 3. Output

```
# universe YYYY-MM-DD
snapshot_at:       # ISO UTC
source:            # cron | chat
scan_filters:      # the macro scan v2 params used
returned:          # number of tokens the screener returned

## snapshot
| mint | ticker | age_h | mcap | liq | vol24 | top10 | chg24 | g5 | g6 | g7 | g8 | sec | a_setup_screen |
|------|--------|-------|------|-----|-------|-------|-------|----|----|----|----|-----|----------------|

## labels (snapshot of <yesterday>)
| mint | ticker | mcap_then | mcap_now | ret24 | liq_ratio | alive | dead |
|------|--------|-----------|----------|-------|-----------|-------|------|

## summary
labeled: · alive: · dead: · unknown: · median_ret24: · a_setup_alive_rate: · others_alive_rate:
```

Full mints in every row. No prose outside the file.

## 4. Write
- **Chat:** one `writeFile` to `/research/universe-YYYYMMDD.md`. "Staged, tap ✅" until the receipt.
- **Cron:** output only one fenced block, first line exactly `STAGED WRITE /research/universe-YYYYMMDD.md`, then the file. Blake lands it with `flush Universe Log`. The labels step reads yesterday's file, so the chain only works if yesterday's flush landed.

## 5. What the lab does with it
After 7 days (about 200 labeled rows), `lab` can test claims like "a_setup_screen tokens stay alive more often than others" against the benchmark of all rows, out of sample on later days. `desk` reports the running alive and dead base rates.

## Budget
At most 43 tool calls: 1 read, 30 snapshots, 1 search, 1 scan, 10 analyze. Output under 7,000 characters.

## Never
Trade. Invent a label from a failed read. Filter the snapshot down to winners.
