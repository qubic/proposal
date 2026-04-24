# Proposal: Upgrade qRWA Smart Contract

## Proposal
Upgrade the existing qRWA Smart Contract on Qubic to the latest reviewed implementation.

## Available Options
Option 0: No, do not approve the qRWA upgrade.

Option 1: Yes, approve the qRWA upgrade.

## Context
qRWA already exists on-chain.

This proposal is an upgrade proposal, not an initial deployment proposal.

## Summary of Changes

### Revenue Pools, 4 Pool Architecture
- **Pool A** (Qubic Mining): incoming revenue from Qubic farm SCs, governance fees deducted first
- **Pool B** (SC Assets): dividend revenue from user wallets, routed via `POST_INCOMING_TRANSFER`
- **Pool C** (BTC Mining): dedicated revenue address with per-pool payout tracking
- **Pool D** (MLM Water): dedicated revenue address with 10 % reinvestment split

Each pool distributes independently: 90 % to QMINE holders, 10 % to qRWA shareholders.

### New Per-Pool Payout Timestamps
- Replaced single `mLastPayoutTime` with four independent timestamps (`mLastPayoutTimePoolA/B/C/D`).
- Added tick-based payout mode (`QRWA_USE_TICK_BASED_PAYOUT`) as configurable alternative to UTC-based scheduling.
- Each pool triggers on a separate hour on Fridays (12:00 / 13:00 / 14:00 / 15:00 UTC).

### Revenue Threshold Guard
- Added `QRWA_MIN_REVENUE_THRESHOLD = 1000 QU`: pools with revenue below this are skipped instead of distributing near-zero amounts.

### Payout History Ring Buffer
- New per-pool ring buffers (`mPayoutsPoolA/B/C/D`, each 16 384 entries) store individual payout records.
- Queryable via `GetLatestPayouts` (function 11) for full auditability.

### Struct Renames (Naming Consistency)
- `RWAAsset` → `QRWAAsset`
- `RWAGovParams` → `QRWAGovParams`
- `RWAGovProposal` → `QRWAGovProposal`
- `RWALogger` → `QRWALogger`

### State Size Optimisation
- `QRWA_MAX_QMINE_HOLDERS` reduced from `2 097 152` (2M, ~563 MB) to `131 072` (128 K, ~37 MB state).

### Asset Poll System Removed
- `AssetReleaseProposal` and related maps (`mAssetPolls`, `mAssetProposalVoterMap`, `mAssetVoteOptions`) removed to reduce state size and scope.

### Loop Safety (while → for)
- All unbounded `while` loops over collections replaced with bounded `for` loops to comply with QPI contract restrictions.

### Epoch Migration Guard
- One-time migration block guarded by `qpi.epoch() <= 211` ensures state initialisation only runs during the known rollout window.

### New Addresses & Configuration
- `mDedicatedRevenueAddress` (Pool C), `mPoolDRevenueAddress` (Pool D), `mPoolARevenueAddress`, `mFundraisingAddress`, `mExchangeAddress` added to `StateData`; fundraising and exchange addresses are excluded from all distributions.

### No New Contract Deployment
- Same contract index (20), same `contract0020.211` state file. This is an in-place upgrade.

## Technical Implementation
Core implementation PR:
https://github.com/qubic/core/pull/820/changes

## Operational Requirement for Computors
Since qRWA was not active previously in production state, a fresh state file is required:

```bash
dd if=/dev/zero bs=30677232 count=1 > contract0020.197
```

## Risk and Compatibility
- In-place upgrade of an existing smart contract.
- Reviewed and tested in the linked core PR.
- Main operational dependency is the fresh state file creation shown above.

## Recommendation
Vote Option 1 to approve the qRWA smart contract upgrade.
