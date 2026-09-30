---
name: desk
description: The desk board and scorecard. Use when Blake says "desk", "board", "briefing", "pulse", "scorecard", "what's open", "what's due", "regime", or "how am I doing". Rebuilds one board file from the run records, the lab, the kill switch, automations, and a live regime read. Reports hit rate, Brier score, and runs due for grading. Never trades.
---

# desk

The desk is **derived**. Runs, reviews, and lab files are the truth; the board is a projection of them. One file, one write, so it fits the one-tap rule.

## 1. Collect (read only)
- `listFiles` with path `/terminal/runs`, then `/terminal/lab`, then `/research` (never bare: the root page is 100 playbook files and truncates). Read every `/terminal/runs/*/run.md` (skip `_TEMPLATE`; if more than 60, read the newest 60 and say so).
- Latest `/research/review-*.md`, if any.
- Every `/terminal/lab/*/exp.md` header (status line only).
- `/terminal/KILL.md` (`active:` line).
- `listAutomations` for status, then `automation runs` for the latest run of each active automation (status and error only).
- Live regime: at most 2 `searchTools` calls to find the Hyperliquid market snapshot and indicators tools; read BTC, ETH, SOL.

Old runs may use the earlier format (`opened_at_utc`, `## gates` with #78 to #81, no `p`). Read what exists; treat missing fields as missing, never guess.

## 2. Scorecard
Definitions, computed from run.md files only:

| Field | Definition |
|-------|------------|
| runs | all runs |
| open | status OPEN or WATCH |
| graded | `result` is HIT, MISS, or INVALIDATED |
| due | not graded and `horizon_end` (or legacy horizon) is before now |
| hit_rate | HIT / graded |
| brier | mean of (p - y)^2 over graded runs with a `p`; y = 1 for HIT, else 0. Also report how many graded runs lack `p` |
| calibration | graded runs with `p` in bins 0-0.2, 0.2-0.4, 0.4-0.6, 0.6-0.8, 0.8-1: count, mean p, realized HIT rate |
| ignore_rate | IGNORE verdicts / runs |
| ignore_hits | IGNORE runs graded HIT (possible missed gains; tests whether gates are too strict) |
| watch_hits | WATCH runs graded HIT / WATCH runs graded |
| a_setups | runs with a_setup yes, and how many graded |

With fewer than 20 graded runs, print `(n<20, not meaningful yet)` next to hit_rate and brier.

## 3. Board
Build this and compare with the current `/terminal/desk/board.md`. If nothing but the timestamp would change, **do not write**; reply "desk unchanged" with the scorecard line.

```
# desk board
updated_at:        # ISO UTC
rung:              # from SOUL (OFF unless Blake changed it)
kill:              # active true | false

## scorecard
runs: · open: · graded: · due: · hit_rate: · brier: · ignore_rate: · ignore_hits: · a_setups:
calibration:       # one line per non-empty bin

## due for grading
- <run_id> · <ticker> · horizon_end · "grade <run_id>"

## open
- <run_id> · <ticker> · verdict · risk · p · horizon_end · invalidation (short)

## regime
BTC: trend · bias · mark (Probable)
ETH:
SOL:
risk_on_off:

## lab
- <exp_id> · status · next

## automations
- <name> · active|paused · last run ok|failed (<error>)

## gaps
- <failed reads, runs missing p, stale review, anything Unknown>
```

One `writeFile` to `/terminal/desk/board.md`. "Staged, tap ✅" until the receipt.

## 4. Reply shape (10 lines max)
Scorecard line, due count with the first two `grade` commands, regime one-liner, any failed automation, then `board staged` or `desk unchanged`.

## Legacy
The ten pane files `/terminal/desk/01-*.md` to `10-*.md` and `/terminal/metrics.md` are superseded by `board.md`. Do not write them. Blake may delete them.

## Never
Write runs, lab, research, or memories. Stage trades. Invent regime numbers when the read fails (write Unknown and list it under gaps).
