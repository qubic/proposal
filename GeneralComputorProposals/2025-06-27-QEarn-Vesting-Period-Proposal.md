# Proposal: QEarn Unlock Vesting Period   

## Purpose

This proposal seeks to adjust the QEarn locking system by introducing a tiered vesting period **after users initiate an unlock request**.
The goal is to prevent short-term manipulation of yield dynamics and ensure a more balanced reward system that favors long-term participation.

---

## Decision to Quorum
Should the QEarn smart contract be updated to include the proposed vesting period logic after unlock requests, as described above?

### Available Options:

> Option 0: NO – Retain current status 

> Option 1: YES – Implement vesting delay based on lock duration  

## Context

QEarn was introduced to reduce the circulating supply of $QUBIC by incentivizing long-term locking through a weekly staking pool model. While the model is functioning as intended on a technical level, recent behavior has revealed imbalances:

- Some users are locking large amounts of $QUBIC, then just exit shortly after, undermining the reward system.
- The ability to unlock immediately (with penalty) has allowed speculative participants to treat QEarn as a short-term farm, rather than a commitment mechanism.

Community feedback has pointed out that this creates an unfair ecosystem and destabilizes weekly yield expectations. This proposal introduces a **post-unlock vesting period** as a mitigation strategy.

---

## Proposal Details

Upon user-triggered unlock, funds will not be immediately released. Instead, a **vesting delay** will be applied based on how long the user has already been locked in the current staking round:

| Lock Duration at Unlock Request | Vesting Delay |
|-------------------------------|---------------|
| Less than 1 month             | 4 weeks       |
| 1–2 months                    | 3 weeks       |
| 3–9 months                    | 2 weeks       |
| 9–12 months                   | No delay – unlock immediately |

Additional points:

- **Final reward share is calculated based on when vesting ends**, not when the unlock request is made.
- **Keeping funds locked beyond vesting has no additional reward impact.**
- **This change will affect all locked funds in QEarn! - New and running locking rounds will be affected by that change**

---

## Rationale

This adjustment discourages speculative short-term locks without fully removing the flexibility to unlock. It:

- Makes reward outcomes more stable and predictable.
- Encourages users to think long-term rather than game weekly lock cycles.
- Disincentivizes farming behavior that undermines reward fairness.
- Protects the integrity of QEarn as a deflationary and commitment-based mechanism.

---

## Implementation Plan

If accepted by the Quorum, the implementation will proceed in two phases:

### Phase 1 – Development
- Development of the necessary QEarn smart contract changes will begin.
- This includes unlock request tracking, vesting timers, and reward calculation logic.
- Frontend elements will be updated to reflect the new behavior (vesting countdowns, unlock availability, reward eligibility messaging).

### Phase 2 – Activation via Smart Contract Proposal
- Once development and internal testing are complete, a **follow-up on-chain proposal** will be submitted to formally activate the new vesting logic in the QEarn smart contract.
- This ensures that no code change occurs without explicit Quorum approval on-chain.


