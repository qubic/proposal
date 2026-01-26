# RandBeacon: Proposal for the Randomness Beacon Contract

This document explains what the `RandBeacon` contract is, why it exists, and how it works. It is written for a mixed audience (developers, operators, and non-programmers) and mirrors the actual implementation in `src/contracts/RandBeacon.h`.

## Summary

RandBeacon is a decentralized randomness beacon based on a commit-reveal scheme. Operators commit to secrets, later reveal them, and the contract combines all revealed secrets into one public random value per round. Users or other contracts can request that random value and pay a fee which is distributed to a deterministic set of winners among the revealers.

Key properties:

- **Manipulation resistance**: No single operator can control the outcome if at least one honest operator reveals.
- **Public verification**: Anyone can recompute the round random value and verify payouts.
- **Economic incentives**: Operators post a refundable deposit and lose it if they fail to reveal.

## Goals

- Provide a **public, auditable** source of randomness for Qubic contracts.
- Make the randomness **difficult to bias** via commit-reveal and penalties.
- Offer a **simple, deterministic** payout mechanism to incentivize participation.

## Participants

- **Operators**: Provide entropy (secret seeds) and reveal them later.
- **Callers (users or contracts)**: Request randomness and pay the usage fee.

## Round Structure (Timing and Phases)

Each round is **60 ticks** long:

- **Commit phase (ticks 0-29)**: operators submit a commitment hash plus deposit.
- **Reveal phase (ticks 30-59)**: operators reveal seed + salt.
- **Finalization (on next round start)**: randomness is computed and the previous round is stored.

Round index is `floor(tick / 60)`.

## Commit-Reveal Details

### Commit

Operators submit:

```
commitHash = K12( seed || salt || roundId )
```

Rules:

- Must be in commit phase.
- Must include at least the required deposit.
- Only one commit per operator per round.
- Total operators per round are capped.

### Reveal

Operators submit the original `seed` and `salt`.

Rules:

- Must be in reveal phase.
- Contract recomputes the hash and compares to the committed one.
- If mismatch: deposit is forfeited to the dev address.
- If correct: deposit is returned, and the operator is marked as revealed.

## Random Value Construction

At round finalization, the contract XORs all revealed seeds:

```
roundRandom = seed1 XOR seed2 XOR seed3 XOR ...
```

- If **fewer than 2 reveals** occurred, randomness requests return `INSUFFICIENT_REVEALS`.

The resulting `roundRandom` is stored and can be queried later.

## Fee and Reward Distribution

Callers request randomness via `GetRandomWithFee(roundId)` and pay a fixed fee.

If rewards are not yet distributed:

- **90%** goes to a reward pool for operators.
- **10%** goes to a fixed DEV address.

Winners are chosen deterministically from the revealed operators:

```
score = K12( roundRandom || operatorAddress )
```

The **lowest scores win**. The contract picks up to **K winners** (or fewer if fewer reveals).
The reward pool is split equally among the winners.

If rewards were already distributed, the full fee is refunded.

## Limits and Parameters (Current Values)

These are the current constants from the implementation:

- Round length: **60 ticks**
- Commit end: **30** (commit is ticks 0-29)
- Operator deposit: **100000**
- Max operators per round: **512**
- Round history size: **256**
- Minimum reveals for usability: **2**
- Winners per round: **20**
- Fee for random request: **200000**
- Fee split: **90% operators / 10% dev**

## Public Interface (User Procedures & Functions)

### Procedures

- `Commit(commitHash)`
  - Returns: success/error + current round id
  - Requires deposit and commit phase

- `Reveal(seed, salt)`
  - Verifies commitment, returns deposit on success

- `GetRandomWithFee(roundId)`
  - Returns the random value and distributes rewards
  - Requires fee and a finalized round with enough reveals

### Functions

- `GetRoundInfo(roundId)`
  - Returns round random, reward pool, reveal count, and flags

- `GetCurrentRound()`
  - Returns current round id, tick-in-round, phase flags, and counts

These allow callers and tooling to determine precisely why an action failed.

## Security Notes and Assumptions

- The beacon is **secure against bias** as long as **at least one** honest operator reveals a secret.
- Revealing late or withholding a reveal is discouraged via deposit forfeiture.
- The XOR construction is simple and efficient; it relies on the unpredictability of at least one seed.

## Code
[Random Beacon](https://github.com/N-010/core/blob/feature/2025-12-09-RandBeacon/src/contracts/RandBeacon.h)
