# Proposal: Deploy QTREAT Smart Contract

Deploy the QTREAT smart contract on Qubic — a new contract for the **QDOGE protocol** that automates dividend payouts, QDOGE staking, NFT-based mining rewards, and a raffle, all funded from a shared treasury.

## Available Options

- **Option 0:** No, do not approve the QTREAT deployment.
- **Option 1:** Yes, approve the QTREAT deployment.

## Context

QTREAT is a **new** smart contract. This is an initial deployment proposal, not an upgrade — the contract does not yet exist on-chain and introduces no changes to any existing contract.

## Summary of Features

### Staking

- Deposit QDOGE (minimum 10,000,000) to earn a share of a fixed **20,000,000 QU/epoch** reward pool for 52 epochs.
- Staking is a single transaction via QX management-rights transfer; unstaking is a single transaction with a 2-epoch delay and automatic release.
- **Progressive bonus (`growthStreak`):** stakers who grow their *total* holdings (staked + wallet) to a new all-time high each epoch earn an increasing reward multiplier, up to **+500‰** at a 20-epoch streak — rewarding genuine accumulation, not wallet-shuffling.
- **Loyalty bonus:** wallets staked at or above 50,000,000 QDOGE for 12 consecutive epochs accrue a QTREAT token bonus (max 4 per wallet), auto-delivered from a pre-funded pool or manually claimable.

### Dividends

- QTREAT token holders and the Qdoge NFT 1.0 collection share deposited QU dividends each epoch, weighted by holdings / NFT count.
- Of every dividend payout, **5%** goes to the contract's shareholders and the remaining **95%** goes to QTREAT holders and NFT holders.
- Excluded addresses (e.g. exchange wallets) can be omitted from the distribution by the admin.

### Mining & Drip

- The Qdoge ASIC (NFT 2.0) collection consists of four part types that can be assembled into a rig and registered on-chain (`RegisterAsic`).
- Registered rig owners earn a **rarity-weighted** share of the mining reward pool each epoch, at an admin-settable, hard-capped rate.
- Rig ownership and part possession are verified live against the on-chain **QBAY** marketplace — sales automatically redirect rewards to the new owner with no admin action.
- Rig owners additionally receive a per-epoch **QDOGE drip**, rarity-weighted, for one year.

### Raffle

- For 52 epochs, eligible stakers are entered into a per-epoch draw seeded by 256 bits of entropy purchased from the **RANDOM** contract (with a chain-state fallback if unavailable). Every wallet has equal odds.

### Custody & Administration

- General-asset custody: the admin can stage, deposit, revoke, and release QX-managed assets into and out of contract custody.

### Interface

8 view functions (`GetStakingInfo`, `GetPhaseInfo`, `GetFunds`, `GetRaffleInfo`, `GetExcludeAddresses`, `GetNftInfo`, `GetMinerInfo`, `GetAsicCatalogInfo`) and 16 procedures (deposits, staking/unstaking, bonus claims, asset custody, ASIC catalog/rig management, mining rate, drip).

### Tests

Every function was verified end-to-end (staking including rejection paths, custody, mining, raffle, progressive/loyalty bonuses, unstaking, NFT dividends) against a 10-wallet local testnet across 30 real epoch transitions. Full run logs — every transaction hash and decoded detail, plus end-of-epoch payout events — are published in the [QTREAT repository](https://github.com/profitphil/QTREAT).

## Technical Implementation

**Developer:** [double-k-3033](https://github.com/double-k-3033)

Contract source and testnet evidence: [https://github.com/profitphil/QTREAT](https://github.com/profitphil/QTREAT)

Core implementation PR: [https://github.com/qubic/core/pull/973/files](https://github.com/qubic/core/pull/973/files)
