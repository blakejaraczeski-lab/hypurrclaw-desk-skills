---
name: desk-automations
description: Writing, testing, fixing, and landing the output of Blake's crons. Use when Blake says "flush <cron name>", "intake stage0", "write a cron", "fix cron", "automation health", "why did <cron> fail", or when an automation fails with AutomationStepOutputLimitError. Covers the cron prompt rules that stay under the 32k step cap, the test-before-schedule loop, landing a cron's STAGED WRITE block into /research, and opening runs from a Stage-0 file.
---

# desk-automations

Platform facts this skill is built on:
- A cron cannot land a file. Its `writeFile` does not leave a ✅ ticket behind. Crons **produce** a fenced block; Blake **lands** it with `flush`.
- A cron run is killed if its model step passes **32,000 characters** (`AutomationStepOutputLimitError`). Tool calls and their results count against the run in ways that are not documented, so budget both.
- A run record keeps only the delivered text. `automation runs` previews cut off at 500 characters; `automation run output` has the full text.
- Automation kinds: plain cron (`createCronAutomation`), scanner, market alert. File-producing jobs must be plain crons; scanners can't be held to fence-only output.
- `updateAutomation` edits in place (stages ✅). `automation now` runs one immediately.

## A. Writing a cron prompt
Every file-producing cron prompt must:
1. Be under **1,500 characters**.
2. State a **tool budget**: "Use at most N tool calls. Do at most 2 searchTools calls; reuse the tool ids."
3. State an **output budget**: "Total output under 6,000 characters."
4. Cap rows: at most 10 candidates, at most 30 universe lines.
5. End with the **output law**: "Output only one fenced block. First line exactly: STAGED WRITE /research/<file>. No prose outside the fence. If there is nothing to write, output only CRON_SUPPRESS."
6. Name no files to read unless needed; never write files; never stage trades or buttons.
7. Be self-contained. Don't rely on skills inside crons until a test shows the cron runner can see them.

### Stage-0 template (plain cron, every 60 min)
```
Stage-0 SOL scan. Read-only. At most 12 tool calls; at most 2 searchTools calls, reuse ids. Output under 6,000 chars.
Universe: Solana tokens, mcap $50k-$5M, liq >= $15k, 24h vol >= $25k, pair age >= 30m.
Gates: not honeypot; buy and sell tax <= 5%; mint and freeze authority off; LP burned or locked (removable fails); top10 ex pool <= 35%; no heavy bundler/sniper/dev cluster; 24h vol not above 50x liq.
Output only one fenced block. First line exactly: STAGED WRITE /research/stage0-YYYYMMDD-HH.md (UTC). Then:
# stage0 YYYY-MM-DD HH:00 UTC
## universe (max 30)
- <full mint> | <ticker> | mcap | liq | first failed gate or PASS
## a_setups (max 10, all gates PASS)
- mint: <full> | ticker: | liq: | mcap: | top10: | lp: | claim: <24h numeric claim> | p: <0-1> | invalidation: <kill>
No prose outside the fence. If the universe is empty, output only CRON_SUPPRESS.
```
The `## universe` section records every eligible token, not just passes, so the lab has an honest denominator.

### Daily Review template (plain cron, 00:00 UTC)
```
Daily close for the previous UTC day. Read-only. At most 15 tool calls; at most 2 searchTools calls. Output under 8,000 chars.
Read /terminal/runs/*/run.md, the previous /research/review-*.md, /terminal/KILL.md, and listAutomations.
Output only one fenced block. First line exactly: STAGED WRITE /research/review-YYYYMMDD.md. Body sections: scorecard (runs, open, graded, due, hit_rate, brier, ignore_rate), day (opened, graded, due, missing_prediction), decisions (at least one decision line, or one no_trade line), lessons (max 3), automations, tomorrow (max 10 ranked tasks), debt.
No prose outside the fence.
```

## B. Test before schedule
1. Create or update the cron (stages ✅; Blake taps).
2. `automation now` on it.
3. `automation runs` for the new run id, then `automation run output` for the full text.
4. Check: finished without error; output is one fence (or CRON_SUPPRESS); the path starts with `/research/`; row caps held; total under the budget.
5. Only then leave it scheduled. If it fails, cut the tool budget or row caps and repeat. Report each test as `test N: ok | failed (<error>) · chars · rows`.

## C. flush <cron name>
1. `listAutomations`; match the name or id exactly.
2. `automation runs` -> latest run id -> `automation run output` (full text, never the 500-char preview).
3. Find exactly one fenced block whose first line is `STAGED WRITE <path>`.
4. Refuse, with the reason, if: no output, no fence, more than one fence, empty body, or `<path>` does not start with `/research/` or contains `..`.
5. Stage one `writeFile` with the body **unchanged**. "Staged, tap ✅" until the receipt.

## D. intake stage0 [YYYYMMDD-HH]
1. Read the named `/research/stage0-*.md` (default: the newest). Missing: refuse and suggest `flush` first.
2. For each `a_setups` row, oldest first: run the `ca-intake` flow with `source: stage0-<stamp>`, using the row's claim, p, and invalidation as the prediction (after fresh live reads confirm the gates still pass; if they don't, the verdict reflects today's reads and the prediction still comes from the row).
3. One run.md write per row, one tap each. If taps stop: `partial: N of M runs opened` and list the remaining mints.

## E. automation health
`listAutomations`, then the last 3 `automation runs` per active automation. Report per automation: status, last 3 results, error names. For `AutomationStepOutputLimitError`: apply section A (tighter budgets), then section B. For provider stream errors, retry once before changing anything. Five consecutive failures auto-pause a cron; resume only after a passing test.

## Never
Stage trades or buy buttons from a cron. Land a file outside `/research/` with flush. Paraphrase a flushed body. Schedule an untested prompt.
