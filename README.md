# ReputationVault

**A reputation-weighted contributor treasury vault, built on programmable
routing primitives on Stacks (Clarity).**

Submitted for: *Programmable Money Flows on Stacks* — FlowVault bounty.

---

## The financial behavior this introduces

Most DAO/startup payroll tools do one of two things: pay a fixed salary on a
schedule, or let an admin manually push funds around. Both break down —
fixed salaries overpay inactive contributors and underpay high performers;
manual routing doesn't scale and re-introduces a trusted intermediary.

ReputationVault instead makes **payout size a function of on-chain
contribution history**, and makes **payout timing a function of automated
epochs (optionally gated by milestones)** — not a person's judgment call.

```
                              DEPOSIT
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   vault-core.clar        │
                    │   auto-split by bps      │
                    └────────────┬─────────────┘
                                 │
        ┌────────────────────────┼─────────────────────────┐
        ▼                        ▼                          ▼
 CONTRIBUTOR POOL         OPERATIONAL RESERVE         EMERGENCY FUND
 (e.g. 70%)               (e.g. 20%)                  (e.g. 10%)
        │                        │                          │
        ▼                        ▼                          ▼
 reputation.clar          admin withdraws            emergency-fund.clar
 scores each                directly, no                time-lock (~1wk)
 contributor from            lock, anytime               + N-of-M multisig
 milestones (+) and                                       approval required
 decays on inactivity (-)
        │
        ▼
 payout-router.clar
 distribute-epoch(dao-id):
   - permissionless trigger, once per epoch (~30 days, block-gated)
   - optionally blocked until milestone-gate.clar reports "unlocked"
   - pays every contributor: share = pool * (their score / total score)
   - rounding dust stays in the pool, rolls into next epoch
        │
        ▼
 CONTRIBUTORS PAID PROPORTIONALLY TO REPUTATION, NOT A FIXED SALARY
```

## Why this satisfies "new financial behavior," not a payment clone

- **Payroll isn't fixed-amount.** It's recomputed every epoch from a live,
  decaying reputation score — an active contributor's *share of the pool*
  grows as inactive contributors' scores decay, with no admin intervention.
- **Automation is real, not cosmetic.** `distribute-epoch` is permissionless
  and block-height gated — any keeper bot or cron job can trigger it. No
  human has to remember to run payroll.
- **Composable primitive.** `reputation.clar`'s score is readable by any
  other contract (`get-reputation`, `get-total-reputation`) — a future
  lending, governance-weight, or reward contract could reuse the same score
  without touching payout logic.
- **Event-triggered unlocks.** `milestone-gate.clar` turns time-based
  payroll into outcome-based payroll: funds only move once a reporter
  attests a milestone was hit, and the flag self-consumes so every epoch
  needs a fresh unlock.
- **Genuine custody separation.** Three pools, three different release
  mechanisms (admin-direct / reputation-weighted-automatic /
  multisig-plus-timelock), enforced at the contract level via
  `contract-caller` checks — not just labels in a UI.

## Contracts

| Contract | Responsibility |
|---|---|
| `vault-core.clar` | DAO registration, split-ratio config, deposit auto-splitting, sole STX custody, gated spend functions per pool |
| `reputation.clar` | Per-DAO contributor registry, milestone-driven score increases, time-based decay, total-reputation aggregation |
| `payout-router.clar` | Epoch-gated, permissionless, reputation-weighted contributor payouts; enforces milestone gate when enabled |
| `milestone-gate.clar` | Optional event-triggered unlock flag per DAO, consumed on each successful gated distribution |
| `emergency-fund.clar` | N-of-M multisig + minimum time-lock proposal/approval/execution flow for the emergency pool |

Authorization pattern used throughout: pool-spending functions in
`vault-core.clar` check `contract-caller` (the immediate calling contract,
not the original tx sender) so only `payout-router.clar` can call
`pay-contributor`, and only `emergency-fund.clar` can call
`release-emergency`. This keeps all STX custody in one auditable contract
while still enforcing per-pool release rules.

## Repo structure

```
reputation-vault/
├── Clarinet.toml
├── contracts/
│   ├── vault-core.clar
│   ├── reputation.clar
│   ├── payout-router.clar
│   ├── milestone-gate.clar
│   └── emergency-fund.clar
├── tests/
│   ├── vault-core.test.ts
│   ├── reputation.test.ts
│   ├── payout-router.test.ts
│   └── emergency-fund.test.ts
├── settings/
│   └── Devnet.toml
├── frontend/                 (minimal demo UI, not the point of the submission)
│   ├── src/App.jsx
│   ├── src/config.js
│   └── ...
├── package.json
├── vitest.config.js
└── tsconfig.json
```

## Running it

This was authored without a local Clarinet install available in the build
environment, so **run a syntax check first**:

```bash
# 1. Install Clarinet (https://github.com/hirosystems/clarinet)
curl -L https://install.clarinet.sh | bash   # or brew install clarinet

# 2. From the project root
clarinet check

# 3. Run the full test suite (Clarinet SDK + Vitest)
npm install
npm test
```

To try it interactively:

```bash
clarinet console
```

```clarity
(contract-call? .vault-core register-dao u7000 u2000 u1000)
(contract-call? .vault-core deposit u1 u100000000)
(contract-call? .reputation add-reporter u1 'ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG)
(contract-call? .reputation record-milestone u1 'ST3AM1A56AK2C1XAFJ4115ZSV26EB49BVQ10MGCS0 u40)
(contract-call? .payout-router distribute-epoch u1)
```

To run the demo frontend against a testnet deployment:

```bash
cd frontend
npm install
# edit src/config.js with your deployed contract address
npm run dev
```

## Test coverage

- `vault-core.test.ts` — split-ratio validation, exact deposit math,
  rounding-dust conservation, admin-gating on withdrawals, cross-contract
  authorization on `pay-contributor` / `release-emergency`.
- `reputation.test.ts` — reporter authorization, milestone accumulation,
  contributor-registry de-duplication, total-reputation aggregation, decay
  timing and magnitude.
- `payout-router.test.ts` — epoch block-height gating, permissionless
  triggering, proportional payout correctness, empty-pool/zero-reputation
  guards, milestone-gate enforcement and flag consumption.
- `emergency-fund.test.ts` — proposal/approval/execution lifecycle,
  time-lock enforcement independent of approvals, threshold enforcement
  independent of time-lock, double-approval prevention, custody isolation.

## Known simplifications (documented, not hidden)

- Reputation decay must be triggered per-contributor by an external
  caller (no native "cron" in Clarity) — this is standard for on-chain
  automation and is designed to be called by a keeper bot or the next
  interested party.
- The contributor registry is capped at 200 principals per DAO
  (`as-max-len? ... u200`) to keep `fold` operations bounded and gas
  predictable; raise the cap for larger orgs if needed.
- Epoch length (~30 days) and decay thresholds (~30 days) are constants
  tuned for a monthly payroll cadence — trivially adjustable before
  deployment for a different cycle.
