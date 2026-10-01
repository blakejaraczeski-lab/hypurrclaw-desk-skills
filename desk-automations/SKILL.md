---
name: desk-automations
description: Writing, testing, and fixing Blake's crons. Use when Blake says "write a cron", "fix cron", "automation health", "test <cron>", "why did <cron> fail", or names a Desk cron (HL Tape, Risk Sentinel, Unlock Radar, Morning Brief, Attention Radar). Covers the sentinel pattern (silent unless something matters), exact tool ids for prompts, schedule formats, and the test loop.
---

# desk-automations

## What a cron is for
A cron is a **Telegram sentinel**, not a file writer. Verified 2026-10-01: every run's full text reaches Blake's Telegram chat, but the stored run record keeps only a short preview (`no_outbox_rows`), and a cron cannot write files. So crons alert, brief, and remind. Files are written from chat, one ✅ each.

Blake's local engine computes signals and grades. It writes `research/desk/book.md` (this week's book, entries, core levels, breaker, new listing) for crons to read. Crons never recompute returns beyond one division per line.

## Writing a prompt
1. Under 1,000 characters for the whole chat message (the web input cap).
2. Name the tools. Start with "No getToolDetails, no searchTools; tools via executeSafeReadTool." Known ids:
   - getHyperliquidMarketSnapshot (one coin: funding APR pct, open interest, mark, prev day price)
   - getHyperliquidExchangeOverview (mids for every market, one call)
   - getHyperliquidCandles, getPerpsMarketRegime
   - getHyperliquidAllPerpMetas (every dex; output gets truncated, so don't rely on it for listing detection)
   - getCoinalyzeFunding (up to 20 coins, per-interval rates, rate-limits), getCoinalyzePositioning
   - macro scan (v1): mode market, chain sol, limit at most 25
   - web_search, open_page, readFile
3. Silence rule: "If nothing to report, reply exactly CRON_SUPPRESS." A scheduled run then sends nothing. A manual run sends an all-clear.
4. Failure rule: report a degraded read in one line only when most reads fail. Never present Unknown as clear.
5. End with "No trades, no writes."
6. Creation phrasing that works: "Create a plain cron automation (createCronAutomation, not a scanner) named X, schedule Y, active, with exactly this prompt: ..."
7. Schedules: "every N minutes" or "daily at HH:MM UTC". Offsets like `30 */4 * * *` are rejected.

## Test loop
1. Create or `updateAutomation` (✅).
2. `runAutomationNow` (✅). Several runs in one message share one ticket.
3. Read the result in Telegram, or `listAutomationRuns` then `getAutomationRunOutput`.
4. Pass: the right lines or CRON_SUPPRESS, real numbers copied from tools, no noise. Fail: name the missing tool or field and fix the prompt, then rerun.

## automation health
`listAutomations`, then the last 3 runs per active automation: status, suppressed or delivered, errors. Five consecutive failures auto-pause a cron; resume only after a passing test.

## Never
Stage trades or wallet actions from a cron. Schedule an untested prompt. Rebuild the old file-producing crons (Stage-0, Universe Log cron, Daily research queue); they are archived.
