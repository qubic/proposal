# Proposal: Upgrade Qraffle Smart Contract

Upgrade the existing qraffle Smart Contract on Qubic to the latest reviewed implementation, adding verifiable on-chain randomness and asset-raffle refinements.

## Available Options

- **Option 0:** No, do not approve the qraffle upgrade.
- **Option 1:** Yes, approve the qraffle upgrade.

## Context

qraffle already exists on-chain. This proposal is an upgrade proposal, not an initial deployment proposal. It builds on the previously approved asset-raffle release.

## Summary of Changes

### Verifiable Randomness (RANDOM SC integration)

- Winner selection now mixes in independent entropy purchased from the **RANDOM smart contract** (collateral tier `0`, `256` bits) at each `END_EPOCH`, strengthening the existing digest-based seed.
- The entropy purchase is **additive and fail-safe**: if RANDOM's pool is empty or the purchase fails, raffle settlement proceeds unchanged using the digest-only seed and the fee is refunded in full — settlement is never blocked.
- Funded by a **1% entropy reserve** carved out of QuRaffle and Asset Raffle Qu pools, **retained in the contract's own balance** (never transferred out), building a self-sustaining reserve over time. Token-denominated token-raffle pools do not fund the reserve.
- To accommodate the 1% entropy reserve, the winner/creator share of a Qu-denominated pool adjusts from **80% to 79%** (the remaining 21%: 5% burn, 5% DAO/register, 8% shareholders, 1% charity, 1% fee, 1% entropy reserve).

### Asset Raffle Prize Eligibility

- Removes the restriction that blocked **QRAFFLE SC shares** and the **QXMR token** from being used as asset-bundle raffle prizes. They may now be included in bundles.

### Parameter Change

- Default QuRaffle pool/entry amount lowered from **10,000,000 Qu to 1,000,000 Qu**.

### Tests

- New coverage for the RANDOM integration: successful entropy purchase with a funded reserve, graceful fallback when RANDOM's pool is empty (full refund, digest-only seed), and no purchase attempted when the reserve is unfunded.

## Technical Implementation

**Developer:** [double-k-3033](https://github.com/double-k-3033)

Core implementation PR: [https://github.com/qubic/core/pull/968](https://github.com/qubic/core/pull/968)
