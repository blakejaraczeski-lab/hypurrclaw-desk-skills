# HypurrClaw desk skills

Skills for a self-scoring research desk on [HypurrClaw](https://hypurrclaw.xyz). The model already knows how to research; these skills add the process: every token gets a prediction before a verdict, every prediction gets graded, and capital stays at zero until the graded record earns it.

Instruction-only. No keys, no wallets, no code. Every file write still needs a ✅ in HypurrClaw.

## Skills

| Skill | Trigger | Writes (one file per use) |
|-------|---------|---------------------------|
| [ca-intake](ca-intake/SKILL.md) | a pasted token address, "dd", "safe?", `grade <run_id>` | `/terminal/runs/<run_id>/run.md` |
| [deep-dd](deep-dd/SKILL.md) | "deep dd", "red team", narrative or team claims | appends `## research` to the run |
| [desk](desk/SKILL.md) | "desk", "board", "pulse", "scorecard", "what's due" | `/terminal/desk/board.md` |
| [daily-review](daily-review/SKILL.md) | "daily review", "EOD", "queue", Daily Review cron | `/research/review-YYYYMMDD.md` |
| [cost-bench](cost-bench/SKILL.md) | "cost bench", "slippage", G9 UNKNOWN | `/terminal/lab/costs-<chain>.md` |
| [lab](lab/SKILL.md) | "lab", "backtest", "is this edge", "promote" | `/terminal/lab/<exp_id>/exp.md` |
| [wallet-dossier](wallet-dossier/SKILL.md) | a wallet address or handle, "who is this wallet" | `/terminal/entities/wallets/<chain>_<address>.md` |
| [desk-automations](desk-automations/SKILL.md) | "flush <cron>", "intake stage0", "fix cron" | lands one `/research/` file per flush |

Global rules (evidence labels, prediction before outcome, "staged, not saved," the capital rung, the kill switch) live in [SOUL.md](SOUL.md). Paste it into Workspace → Identity; it is not a skill.

## Install
In HypurrClaw, open Browse skills and paste one folder URL at a time:

```
https://github.com/blakejaraczeski-lab/hypurrclaw-desk-skills/tree/main/<skill>
```

Suggested order: `ca-intake`, `desk`, `daily-review`, `desk-automations`, `cost-bench`, `lab`, `deep-dd`, `wallet-dossier`.

## The loop
```
paste CA -> ca-intake (gates, prediction, run.md)
         -> horizon passes -> grade <run_id> (outcome)
         -> desk (scorecard: hit rate, Brier, calibration, due)
         -> daily-review (decision or no-trade line, lessons, tomorrow's queue)
cost-bench (read-only quotes) -> G9 and net-of-cost lab results
lab (graded runs as data, frozen rule, untouched OOS) -> PASSED -> Blake may move the rung
```

## Design rules
- **One file per use.** HypurrClaw allows one pending write at a time, so each skill stages exactly one `writeFile`.
- **Derived, not duplicated.** Runs are the truth. The board and reviews are projections of runs; nothing copies a rule into another file.
- **Cheap discovery.** Each skill caps its `searchTools` calls, because tool discovery was the biggest hidden cost.
- **Crons produce, chat lands.** Cron output is one `STAGED WRITE` fence; `flush` lands it.
- **Capital rung OFF** until a lab experiment passes and Blake changes SOUL.

## Replaces
These skills replace the earlier file-based doctrine: the numbered playbooks, `/terminal/packs/*`, `methodology.md`, `FACTORY.md`, `00-INDEX.md`, desk panes `01` to `10`, `metrics.md`, `/research/queue-README.md`, and the legacy journals. Those files can be removed once the skills are installed and tested.
