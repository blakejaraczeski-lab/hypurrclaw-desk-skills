---
name: deep-dd
description: Deep due diligence and red-team review on a token that already has a run record. Use when Blake says "deep dd", "red team", "full research", "case against", "why is <token> moving", or makes a narrative, team, partnership, developer, or wallet-flow claim about a token. Adds a research section to the token's run.md. Never changes the prediction and never buys.
---

# deep-dd

`ca-intake` answers "is it safe and tradeable right now." This skill answers "is the story true, and what is the strongest case against it." It extends an existing run; it never opens one.

## 0. Preconditions
- Call `listFiles` with path `/terminal/runs` (never bare: the root page returns 100 playbook files first and truncates); open the latest run whose folder ends with the mint's first 8 characters and confirm the full mint. None: run `ca-intake` first, then come back.
- Tool budget: at most **6** `searchTools` calls and **12** reads total. Stop early when the verdict is clear.

## 1. Narrative to metric
List each claim being made about the token (from Blake, socials, the project site). For each, name the metric that would prove it and check it:

| Claim type | Needs |
|------------|-------|
| Adoption | users, transactions, fees, retention, independent integrations |
| Developer momentum | releases, merged PRs, active contributors, shipped features |
| Liquidity | depth and exitability at size, not headline volume |
| Community | retention and organic participation, not follower count |
| Partnership | an announcement from the partner's own channel |

A claim with no measurable metric is **unverified marketing**.

## 2. Sources
- Primary first: contract, explorer, official docs, the project's repo, the partner's own account.
- Social: `x_keyword_search` / `x_semantic_search` / `x_thread_fetch`. Treat posts as REPORTED CLAIMS; note who said it and whether they are paid or anonymous.
- Web: `web search`, `open_page`. Note publication time; stale dashboards count as Unknown.

## 3. Developer signal (if a repo exists)
Release cadence, merged PRs, contributor concentration, CI status, security notices. Discount generated files, formatting commits, and forks. Code activity can support a product thesis; it never validates price or proves security.

## 4. Wallet flows (if relevant)
Deployer, treasury, LP, and top-holder movements. For each: amount relative to supply and liquidity, destination (CEX, LP, contract), timing. State at least one innocent explanation. Never infer intent without corroboration. Deep wallet work goes to `wallet-dossier`.

## 5. Adversarial check
For each critical input ask: who benefits if it is false, and how cheaply could it be faked? Downweight cheap inputs: wash volume, sybil holders, bought engagement, paid callers, edited announcements, impersonated accounts, screenshots.

## 6. Red team
Write the strongest case **against** acting, as if the thesis is wrong: contract risk, concentration, economic flaws, dependencies, liquidity traps, suspicious flows, fake partnerships, weak sources, alternative explanations for any price move. If you cannot state why not to act, the research is not done.

## 7. Causal labels
Any "X caused Y" gets one label: Unsupported, Plausible mechanism only, Observational association, Quasi-experimental, or Strong evidence. Most crypto claims stay at association.

## 8. Save
Rewrite the run's `run.md` with every existing section copied **exactly** and a new section appended:

```
## research
researched_at:        # ISO UTC
claims:               # claim -> metric -> finding -> label, one line each
sources:              # primary sources used, one line each
dev_signal:           # or n/a
flows:                # or n/a
cheap_inputs:         # inputs downweighted and why
case_against:         # 3 to 6 lines, strongest first
causal:               # claim -> label
verdict_change:       # none | WATCH -> IGNORE | ... (never -> TRADE here)
```

One `writeFile`. "Staged, tap ✅" until the receipt.

- The prediction is sealed. If the research shows the thesis itself was wrong, say so and suggest a **new run** after this one is graded.
- The verdict may drop (WATCH to IGNORE). It never rises to TRADE from this skill.

## Reply shape (15 lines max)
Verdict change first, then the top 3 lines of `case_against`, then any claim labeled unverified marketing, then `run.md staged, tap ✅`.
