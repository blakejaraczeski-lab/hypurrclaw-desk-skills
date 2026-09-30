---
name: wallet-dossier
description: Evidence-only profile of a wallet or trader. Use when Blake pastes a wallet address (not a token), a FOMO or KOL handle, or says "who is this wallet", "dossier", "track this wallet", "is this smart money", or "what did this wallet do". Labels every attribution Verified, Probable, Possible, or Unknown, never doxxes, and saves one dossier file per wallet. Never copies trades.
---

# wallet-dossier

Wallet flows are context, not a buy or sell signal. This skill records what can be observed about a wallet and how confident each label is.

## 1. Resolve
- Full address and chain. If the address is a token mint, hand off to `ca-intake`.
- A handle: use the FOMO trader tool (via `searchTools`, then `executeSafeReadTool`) to get linked wallets. The link itself is a label with its own confidence.
- Tool budget: at most 4 `searchTools` calls and 10 reads.

## 2. Observe
| Field | Notes |
|-------|-------|
| first / last activity | timestamps |
| balances, material holdings | tool-verified only |
| role | deployer, treasury, LP, CEX, market maker, bot, trader, unknown |
| linked contracts | deployed or heavily used |
| patterns | timing, size, venues, hold time |
| notable counterparties | with their labels |
| net flows | 7d and 30d if readable |
| risk events | rugs deployed, dumps into buyers, sanctioned or flagged interactions |
| related wallets | with the evidence that links them |

## 3. Label every attribution
- **Verified**: on-chain proof or the owner's own public statement.
- **Probable**: several independent signals agree.
- **Possible**: one weak signal, such as timing or clustering.
- **Unknown**: no evidence.

Never claim a real-world identity from clustering. No personal details beyond what the wallet owner has made public. "Smart money" is a performance claim: it needs a win rate over a stated sample, not a label.

## 4. Performance (only if the data exists)
Win rate, median hold, and realized PnL over a stated sample and window, with the source. Say "sample too small" under 20 trades. Past wins are not a reason to copy.

## 5. Save
One `writeFile` to `/terminal/entities/wallets/<chain>_<address>.md`. If the file exists, rewrite it with a `changelog` line and keep prior labels unless new evidence changes them.

```
# wallet <chain>:<address>
refreshed_at:
role:             # label · confidence
handle:           # label · confidence · source, or none

## activity
first_seen:
last_seen:
holdings:         # tool-verified, with read time
patterns:
net_flows:

## links
contracts:
counterparties:
related_wallets:  # address · relation · confidence · evidence

## risk
events:

## performance
sample:
win_rate:
median_hold:
source:

## runs
- <run_id>        # runs where this wallet mattered

## changelog
- <ISO> · <what changed and why>
```

"Staged, tap ✅" until the receipt.

## Reply shape (10 lines max)
Role with confidence, 3 most important observations, any risk event, the performance line or "sample too small," then `dossier staged, tap ✅`.

## Never
Set up copy trading or stage trades (copy-trade watchers need Blake's explicit request, outside this skill). Dox. Upgrade a label without new evidence.
