# ReputationVault — Submission Writeup

## Project Name
ReputationVault

## One-line tagline
A treasury vault where contributor pay is automatically weighted by
on-chain reputation, not fixed salaries — built on programmable routing
primitives on Stacks.

## The problem
Most DAO and startup payroll tools do one of two things: pay a flat salary
on a schedule, or let an admin manually push funds around by hand. Both
break down. Fixed salaries overpay inactive contributors and underpay high
performers. Manual routing doesn't scale and quietly reintroduces a
trusted middleman into a system that was supposed to remove one.

## The solution
ReputationVault makes payout size a function of on-chain contribution
history, and makes payout timing a function of automated, optionally
outcome-gated epochs — not a person's judgment call.

Every deposit into the vault is instantly and automatically split three
ways:
- **70% → Contributor Pool** — distributed each epoch, weighted by each
  contributor's live reputation score
- **20% → Operational Reserve** — spendable directly by the DAO admin for
  day-to-day costs
- **10% → Emergency Fund** — locked behind a time-lock and a multisig
  approval process, for true emergencies only

Reputation scores rise when an authorized reporter logs a completed
milestone, and quietly decay if a contributor goes inactive — so the
system self-corrects over time without anyone manually rebalancing pay.

Distribution is triggered by anyone, permissionlessly, once per epoch —
no admin has to remember to run payroll. DAOs can optionally require a
milestone to be unlocked before that epoch's payout can go out at all,
turning fixed-schedule payroll into outcome-based payroll.

## Why this is a new financial behavior, not a payment clone
- Payroll isn't a fixed amount — it's recomputed every epoch from a live,
  decaying reputation score.
- Automation is real: `distribute-epoch` is permissionless and
  block-height gated, so a keeper bot (or literally anyone) can trigger
  it — no human has to remember.
- The reputation score is a reusable, composable primitive. Any other
  contract can read `get-reputation` or `get-total-reputation` and build
  on top — a future governance-weight or lending contract could reuse the
  same score without touching payout logic at all.
- Milestone gating turns time-based payroll into event-triggered payroll:
  funds only move once an authorized reporter attests a milestone was
  actually hit, and the flag consumes itself so every epoch needs a fresh
  unlock.
- Custody is genuinely separated by rule, not just by label in a UI: three
  pools, three different release mechanisms (admin-direct /
  reputation-weighted-automatic / multisig-plus-timelock), enforced at
  the contract level.

## Architecture
Five Clarity contracts, each with a single responsibility:

| Contract | Responsibility |
|---|---|
| `vault-core.clar` | DAO registration, split-ratio config, deposit auto-splitting, sole STX custody, gated per-pool spend functions |
| `reputation.clar` | Per-DAO contributor registry, milestone-driven score increases, time-based decay, total-reputation aggregation |
| `payout-router.clar` | Epoch-gated, permissionless, reputation-weighted contributor payouts; enforces milestone gate when enabled |
| `milestone-gate.clar` | Optional event-triggered unlock flag per DAO, consumed on each successful gated distribution |
| `emergency-fund.clar` | N-of-M multisig + minimum time-lock proposal/approval/execution flow for the emergency pool |

All STX custody lives in a single contract (`vault-core`), which only
releases funds to two other contracts it explicitly trusts —
`payout-router` for contributor pay and `emergency-fund` for emergency
releases — enforced via `contract-caller` checks. This keeps the system
auditable in one place while still enforcing different release rules per
pool.

## What's tested
36 automated tests across 4 test suites, covering: split-ratio validation,
exact deposit-splitting math (including rounding-dust conservation),
admin-only vs. permissionless access control, reputation accumulation and
decay timing, proportional payout correctness, epoch gating, milestone-gate
enforcement, and the full multisig + time-lock proposal lifecycle for the
emergency fund. All 36 pass.

## Live deployment (Stacks Testnet)
- Deployer address: `ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG`
- All contracts: https://explorer.hiro.so/address/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG?chain=testnet
- `vault-core`: https://explorer.hiro.so/txid/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG.vault-core?chain=testnet
- `reputation`: https://explorer.hiro.so/txid/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG.reputation?chain=testnet
- `payout-router`: https://explorer.hiro.so/txid/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG.payout-router?chain=testnet
- `milestone-gate`: https://explorer.hiro.so/txid/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG.milestone-gate?chain=testnet
- `emergency-fund`: https://explorer.hiro.so/txid/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG.emergency-fund?chain=testnet

## Tech stack
Clarity smart contracts, Clarinet SDK + Vitest for testing, minimal
React/Stacks.js demo frontend to exercise the flows (deposit, trigger
epoch payout).

## What's next
- Governance-weight integration: let any contract read `reputation.clar`
  scores to weight DAO votes, not just pay
- Configurable decay curves per DAO (linear vs. exponential)
- On-chain milestone verification via an oracle instead of a trusted
  reporter, for fully trustless outcome-gating
