---
name: ca-intake
description: Research record for any token contract address. Use whenever Blake's message contains a Solana base58 mint (32 to 44 chars) or an EVM 0x address, or asks "safe?", "dd", "research", or "buy?" about a token, or says "grade <run_id>". Runs the safety and liquidity gates, writes a prediction before the verdict, and saves one run.md record per token. Never buys.
---

# ca-intake

Turns a pasted contract address into a graded record. The built-in token safety rules still apply; this skill adds the gates, the prediction, and the record. Global rules (labels, staged vs saved, capital rung) live in SOUL.

## 1. Resolve
- Take the full address from the message. Confirm it resolves to a **token** on a chain.
- If it is a wallet, hand off to `wallet-dossier`. If it is a tx signature or nothing, say so in one line and stop.
- Chain slug: `sol`, `base`, `eth`, `arb`, `bsc`, or `hood`.

## 2. Check for an open run
- Call `listFiles` with path `/terminal/runs` (never bare: the root page returns 100 playbook files first and truncates), then read the `run.md` whose folder name ends with this mint's first 8 characters. Confirm the full mint in its header.
- Same mint, opened less than 24h ago, status OPEN or WATCH: **reopen** that run. Refresh the live reads and the verdict. Never change its prediction.
- Otherwise start a **new run**. `run_id` = `YYYYMMDD-HHMMSS-<chain>-<first 8 chars of mint>` in UTC. If an older run exists for the mint, put its id in `continue_of`.
- If the check itself fails (list or read error), start a new run with `continue_of: unknown`. Never skip the record because the check failed; an unrecorded IGNORE is lost data.

## 3. Live reads
Use these catalog tools (all via `executeSafeReadTool`). Find their ids with **one** `searchTools` call, query `analyze token gmgn token security market token snapshot gmgn token top holders`, then reuse them. No other research in this skill; `deep-dd` does that.
- `analyze token` (address, chain) and `gmgn token security` (chain, address): honeypot, tax, mint and freeze authority, LP.
- `market token snapshot` (address, chain): price, mcap, liquidity, volume, pair age, venue.
- `gmgn token top holders` (chain, address) or `token top holders` (mint): concentration and tags.

If `analyze token` leaves authority or tax blank, try `gmgn token security` before marking UNKNOWN.
- Security: honeypot, buy and sell tax, mint authority, freeze authority, LP lock or burn.
- Market: price, mcap, liquidity, 24h volume, pair age, venue.
- Holders: top 10 share excluding pool and vault; bundler, sniper, and dev tags.

A read that fails or comes back partial is Unknown.

## 4. Gates
Mark each gate PASS, FAIL, or UNKNOWN. UNKNOWN is never a pass.

| # | Gate | PASS when |
|---|------|-----------|
| G1 | Contract | Full mint resolves on the stated chain |
| G2 | Honeypot and tax | Not a honeypot; buy and sell tax each 5% or less |
| G3 | Authorities | Mint and freeze authority both disabled |
| G4 | LP | Burned or locked with proof; removable LP fails |
| G5 | Liquidity | $15k or more |
| G6 | Holders | Top 10 (ex pool and vault) 35% or less; no heavy bundler, sniper, or dev cluster |
| G7 | Activity | 24h volume $25k or more; pair age 30 minutes or more; volume not wildly out of line with liquidity (flag wash if 24h volume is above 50x liquidity) |
| G8 | Band | Mcap $50k to $5M. Outside the band is not a fail; it only blocks the A-setup label |
| G9 | Costs | Round-trip cost at the intended size is below the expected move. Read `/terminal/lab/costs-<chain>.md` (built by `cost-bench`) and use the row for this liquidity bucket. No file, or no row for the bucket: UNKNOWN |

**A-setup** = G1 to G8 all PASS. Say "A-setup" in the verdict line when true.

## 5. Prediction, then verdict
Write the prediction first:
- `claim`: one falsifiable sentence about this token over the horizon, stated with numbers (for example "mcap stays at or above $X and liquidity at or above $Y through horizon_end").
- `p`: probability the claim holds, 0 to 1. Be honest; 0.5 is fine.
- `invalidation`: the observable that kills it early.
- `horizon_end`: exact UTC time, 24h after `opened_at` unless Blake says otherwise.

Then the verdict:
- **IGNORE**: any of G1 to G4 FAIL, or two or more gates UNKNOWN.
- **WATCH**: G1 to G4 PASS and the token is worth tracking.
- **TRADE**: only if G1 to G9 all PASS **and** the SOUL capital rung allows it. At rung OFF, TRADE is unavailable; say so if Blake asks.
- Risk: Low, Medium, High, or AVOID. Honeypot, tax above 5%, or live mint or freeze authority means AVOID.

## 6. Save the record
Stage exactly **one** `writeFile` to `/terminal/runs/<run_id>/run.md` using this template. No desk, journal, entity, or memory writes. Say "staged, tap ✅" until the receipt arrives.

```
# run <run_id>

## header
run_id:
mint:
chain:
ticker:          # label only
opened_at:       # ISO UTC
status:          # OPEN | WATCH | IGNORE | CLOSED
verdict:         # TRADE | WATCH | IGNORE
risk:            # Low | Medium | High | AVOID
a_setup:         # yes | no
source:          # chat | stage0-YYYYMMDD-HH
continue_of:     # prior run_id or empty

## prediction
claim:
p:
invalidation:
horizon_end:     # ISO UTC

## gates
G1 contract:
G2 honeypot_tax:
G3 authorities:
G4 lp:
G5 liquidity:
G6 holders:
G7 activity:
G8 band:
G9 costs:
reads:           # key numbers at open, each with Verified/Probable/Unknown

## outcome
resolved_at:
result:          # HIT | MISS | INVALIDATED
observed:        # numbers that decided it
notes:
```

On a reopen, rewrite the whole file with the prediction section copied **exactly**, and add a `refreshed_at:` line under header.

## 7. Reply shape (12 lines max)
1. `IGNORE · High · <ticker> <full mint>` (add `· A-setup` when true)
2. `G1 PASS · G2 PASS · G3 PASS · G4 FAIL · ...`
3. Prediction: claim, p, invalidation, horizon end.
4. `run.md staged, tap ✅`

## 8. Grading
When Blake says `grade <run_id>` (or `grade due`), or asks about a token whose run is past `horizon_end`:
- Read the run. Take fresh live reads. Fill `## outcome` only: `resolved_at`, `result`, `observed`. Set status CLOSED.
- HIT: claim held through horizon_end. MISS: it did not. INVALIDATED: the invalidation printed first.
- Grade IGNORE runs too; they are the no-trade record and show whether the gates are too strict.
- Never edit the prediction or gates. One `writeFile` per run graded. For `grade due`, grade oldest first and report "partial: N of M graded" if taps stop.

## Never
- Buy, sell, size, or stage a trade preview.
- Write `/terminal/desk/`, `/research/`, or memories.
- Change a sealed prediction, or fill an outcome before horizon_end or the invalidation.
