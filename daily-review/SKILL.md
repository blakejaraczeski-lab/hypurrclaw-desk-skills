---
name: daily-review
description: End-of-day close and next-day research queue. Use when Blake says "daily review", "EOD", "close the day", "review <date>", "queue", or "what should I research tomorrow", and in the Daily Review cron. Writes one dated review file with a decision or no-trade line, the scorecard, runs due for grading, lessons, and a ranked queue of up to 10 tasks. Never trades.
---

# daily-review

One file per UTC day: `/research/review-YYYYMMDD.md`. It is both the close for the day and the queue for the next one, so there is no separate queue file.

## 1. Day key
- Cron at 00:00 UTC, or chat with no date after 00:00 UTC: close the **previous** UTC day.
- Chat with a date: close that day.
- If the file for that day already exists, rewrite it and add a `refreshed_at:` line. Never write two files for one day.

## 2. Collect (read only)
- `listFiles` with path `/terminal/runs` (never bare), then every `run.md`. Note which were opened, reopened, or graded during the day (by `opened_at`, `refreshed_at`, `resolved_at`).
- The previous day's review, for its `tomorrow` queue.
- `/terminal/lab/*/exp.md` status lines.
- `/terminal/KILL.md`.
- `listAutomations` plus the latest `automation runs` result for each (status and error only).

## 3. Gate
If any run with status OPEN or WATCH has an empty `claim` or `p`, list those runs under `missing_prediction`. Still write the file; the gap is the finding.

## 4. Compose
Use the scorecard definitions from the `desk` skill (runs, open, graded, due, hit_rate, brier, ignore_rate, ignore_hits).

Decision vs no-trade: the day gets **at least one** line in `## decisions`:
- A decision is any WATCH or TRADE verdict, rung change, lab status change, or rule change made that day.
- If there were none, write one `no_trade:` line summarizing the day's IGNOREs and passes, with reasons (liquidity, authority, concentration, wash, weak evidence, costs, regime, no approval).

Lessons: at most **3**, each tied to evidence in a run or grade. A lesson may propose a gate change, but it becomes a rule only when Blake approves it. Log it; don't apply it.

Queue for tomorrow: at most **10** tasks. Score each 0 to 5 on info value, risk urgency, sources ready, hypothesis fit, and reversibility; rank by the sum. Good tasks: grade a due run, verify an authority, check a deployer transfer, compare liquidity before and after an event, run `cost-bench` on a bucket with no data, re-check an UNKNOWN gate. No broad "scan the market" tasks. Unfinished tasks from yesterday's queue carry over with their age.

## 5. File

```
# review YYYY-MM-DD
closed_at:         # ISO UTC
day_utc:
source:            # chat | cron
rung:
kill:

## scorecard
runs: · open: · graded: · due: · hit_rate: · brier: · ignore_rate: · ignore_hits:

## day
opened: <run_ids>
graded: <run_id result, ...>
due: <run_ids with "grade <run_id>">
missing_prediction: <run_ids or none>

## decisions
- <ISO> · <run_id or scope> · <decision> · <reason> · reverse_if: <condition>
- no_trade: <summary>        # only when there were no decisions

## lessons
- <lesson> · evidence: <run_id> · proposed_rule: <or none>

## automations
- <name> · ok | failed (<error>) | paused

## tomorrow
| rank | task | info | risk | sources | fit | reversible | sum | carried_days |
|------|------|------|------|---------|-----|------------|-----|--------------|

## debt
- <repeated UNKNOWN gates, failed reads, missing cost data, with age in days>
```

## 6. Write
- **Chat:** stage one `writeFile` to `/research/review-YYYYMMDD.md`. "Staged, tap ✅" until the receipt.
- **Cron:** a cron cannot land a file. Output **only** one fenced block whose first line is `STAGED WRITE /research/review-YYYYMMDD.md`, then the file body. No prose outside the fence. Blake lands it with `flush Daily Review` (see `desk-automations`).

## Reply shape (chat, 10 lines max)
Day, scorecard line, the decision or no-trade line, due count, top 3 queue tasks, then `review staged, tap ✅`.

## Never
Grade runs (list them as due; `ca-intake` grades). Write desk, runs, lab, or memories. Invent outcomes, fills, or PnL.
