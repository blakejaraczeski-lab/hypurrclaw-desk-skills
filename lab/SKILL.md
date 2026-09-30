---
name: lab
description: Experiments, backtests, and promotion decisions. Use when Blake says "lab", "experiment", "backtest", "test this rule", "is this edge", "freeze", "promote", "kill exp", or names an exp id. Uses graded run records as the dataset, requires a cost model and an untouched out-of-sample window before any edge claim, and writes one exp.md per experiment. Recommends rung changes; only Blake makes them.
---

# lab

A backtest generates hypotheses; it does not prove edge. The lab's dataset is the desk's own graded runs (shadow results: predictions made before outcomes, graded after). One file per experiment: `/terminal/lab/<exp_id>/exp.md`.

## Ids
`exp-NNNN-<slug>`, numbered one higher than the highest existing `exp-*` folder. Killed ids are never reused or rescued; fork a new id.

## Status ladder
`DESIGN -> FROZEN -> RUNNING -> OOS_REVIEW -> PASSED | FAILED`, or `KILLED` at any point.
PASSED means "eligible to recommend a rung change." It is not permission to trade.

## 1. Open (DESIGN)
Write the hypothesis before looking at any results:
- `claim`: the rule and the edge, e.g. "A-setups graded HIT more often than non-A-setups that pass G1 to G4."
- `universe`: which runs count (chain, date range start, source).
- `rule`: exact filter on run fields (gates, verdict, a_setup, p range).
- `label`: what counts as success (HIT; or a numeric outcome from `observed`), and the horizon.
- `benchmark`: what it must beat. Default: all runs in the universe that pass G1 to G4. Alternatives: all runs, or IGNORE runs.
- `economic_why`: the mechanism, stated as a hypothesis.
- `kill_if`: what ends it.

## 2. Freeze (DESIGN -> FROZEN)
All must be true, checked against real files:
- [ ] claim, universe, rule, label, benchmark written above
- [ ] cost model exists: `/terminal/lab/costs-<chain>.md` has 3 or more samples in each bucket the universe uses
- [ ] split is chronological: in-sample window ends before the out-of-sample window starts; OOS runs are graded **after** `frozen_at`, or were never looked at
- [ ] minimum sizes set: at least 30 graded in-sample runs, and 20 graded OOS runs before review
- [ ] tests_run: count of every exp file ever opened (including killed); record it (multiple-testing control)

Any box unchecked: stay DESIGN, or KILL with a reason if it can't be met. Set `frozen_at`. After freezing, the rule, label, and benchmark never change.

## 3. Run (RUNNING)
Compute from graded run.md files only:
- rule group vs benchmark: n, hit rate, Brier, and the difference
- net of costs: if `observed` has entry and exit values, subtract the bucket median round trip; otherwise report hit rate only and say net is unknown
- robustness (report each): split by week, by liquidity bucket, by regime at open (from `reads` or the desk board), and with the p threshold nudged up and down
- **An effect that disappears under a small change is fragile.** Record it as fragile.

## 4. OOS review (OOS_REVIEW -> PASSED or FAILED)
When OOS reaches the minimum n:
- PASSED only if the rule beats the benchmark on OOS, after costs where measurable, is not fragile, and has a written reason it may persist.
- Otherwise FAILED. **No rescue:** do not change the rule and re-test on the same OOS. Fork a new exp.

## 5. Recommend
A PASSED exp may recommend one rung step (OFF -> SHADOW -> MICRO -> EARNED) with the evidence lines. The rung changes only when Blake writes it in SOUL. MICRO means $5 to $10 per position, no leverage, one component tested at a time; pause if live results diverge from shadow.

## File

```
# <exp_id>
status:
opened_at:
frozen_at:
tests_run:
next:

## hypothesis
claim:
universe:
rule:
label:
benchmark:
economic_why:
kill_if:

## freeze
checklist:        # the 5 boxes with [x] or [ ] and evidence
costs_file:
in_sample:        # window + n graded
oos:              # window + n graded

## results
rule_group:       # n · hit rate · brier
benchmark:        # n · hit rate · brier
difference:
net_of_costs:     # value or "unknown: observed lacks entry/exit"
robustness:       # one line per check
fragile:          # yes | no

## decision
verdict:          # PASSED | FAILED | KILLED | pending
reason:
recommendation:   # rung step or none

## changelog
- <ISO> · <event> · <note>
```

One `writeFile` per turn. "Staged, tap ✅" until the receipt.

## Legacy
`exp-0001-stage0-a-setup` stays KILLED (no costs, no OOS). Read it; don't edit it. The old `promote-gate` and `lab-experiment` packs are replaced by this skill.

## Never
Stage trades or drafts. Change a frozen rule. Report results without the benchmark. Claim edge from in-sample alone.
