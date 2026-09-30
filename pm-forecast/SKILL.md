---
name: pm-forecast
description: Shadow forecasting against Polymarket prices, no bets. Use when Blake says "pm", "polymarket", "forecast", "pm grade", "beat the market", or asks for a probability on a real-world event. Picks liquid markets resolving soon, records the desk's own probability next to the market price before resolution, and later scores both with Brier. Measures whether the desk's probabilities beat the market before any money is involved.
---

# pm-forecast

The whole desk is a calibration machine: a probability, a horizon, a grade. Polymarket turns that directly into a benchmark, because the market price is the crowd's probability. If the desk's Brier score can't beat the market's on the same questions, there is no edge to size. This skill never places orders.

## Jurisdiction
Shadow only. Before any real position, Blake must confirm Polymarket access is legal where he is. This skill never checks eligibility or sets up trading.

## 1. Pick markets (`pm new`)
- One `searchTools` call for the ids of `polymarket events`, `polymarket market snapshot`, `polymarket orderbook`, and `polymarket price history`. Reuse them.
- Choose up to **8** markets that: resolve within 14 days, have a two-sided book with a spread of 4 points or less, and have a question with an objective resolution source. Skip sports parlays, markets resolving on a person's statement, and markets already at 2% or below, or 98% or above.
- For each, read the snapshot and orderbook. Record `mid` (between best bid and ask), `spread`, `volume`, `end_date`.

## 2. Forecast (before looking at price history)
For each market, write the desk's own `p` (probability of YES) **with a one-line reason and the base rate used**, then compare with `mid`:
- `edge` = p - mid (in points)
- Flag `candidate` only when |edge| is at least 8 points **and** larger than 2x the spread.
- Use `web search` / `open_page` for primary sources (official schedules, filings, polls with methodology). No more than 2 sources per market. Label each claim Verified, Probable, Possible, or Unknown.

## 3. Save
One `writeFile` per day to `/research/pm-YYYYMMDD.md`:

```
# pm forecasts YYYY-MM-DD
made_at:
n: 

| id | question | end_date | mid | spread | p | edge | candidate | base_rate | reason | resolved | outcome | brier_desk | brier_market |
|----|----------|----------|-----|--------|---|------|-----------|-----------|--------|----------|---------|------------|--------------|
```

`resolved`, `outcome`, and the two Brier columns stay empty until grading.

## 4. Grade (`pm grade`)
- `listFiles` with path `/research` and read every `pm-*.md` with unresolved rows whose `end_date` has passed.
- For each, read the market. Once resolved: `outcome` 1 for YES, 0 for NO. `brier_desk` = (p - outcome)^2, `brier_market` = (mid - outcome)^2.
- Rewrite that day's file with only those columns filled. One `writeFile` per file.

## 5. Scoreboard (in the reply, and for `desk`)
Across all graded rows: n, mean brier_desk, mean brier_market, and the difference; the same for `candidate` rows only. With fewer than 50 graded rows, say `(n<50, not meaningful yet)`.

The sleeve earns a rung step only when the candidate rows beat the market's Brier over 50 or more graded rows **and** the gap is larger than the round-trip spread. `lab` makes that call; Blake changes the rung.

## Reply shape (12 lines max)
Markets picked, the candidates with p vs mid and edge, then `pm file staged, tap ✅`.

## Never
Place, preview, or stage a Polymarket order. Look up price history before writing `p` (that leaks the answer). Forecast markets with no objective resolution.
