# Smart Contract - GGWP (WolfPack)

## Proposal Statement

Allow deployment of the GGWP (WolfPack) smart contract on the Qubic network, enabling automated weekly revenue distribution, staking, and on-chain shareholder governance for WP token holders, SC shareholders, and clan members.

**Available Options:**
- Option 0: No - Do not deploy GGWP
- Option 1: Yes - Deploy GGWP with revenue distribution, staking and shareholder governance

---

## Executive Summary

GGWP (WolfPack) is a non-custodial, deterministic smart contract that programmatically distributes recurring revenue directly to on-chain stakeholders without oracles or admin control over payouts. It is designed to be **self-sustaining** under upcoming execution fees and **governed by its own shareholders**.

**Key Capabilities:**
- Automated weekly revenue distribution at the epoch boundary (`END_EPOCH`) with a 70/10/10/9/1 split
- Token holder distribution (WP token snapshot at `BEGIN_EPOCH`, with a dust threshold)
- Shareholder dividends paid via the protocol's `qpi.distributeDividends()` (676 IPO shares)
- Clan member rewards (weighted by 6 rank tiers: 1.0x to 4.0x multipliers)
- Reinvestment fund (governable destination for ecosystem growth)
- Staking mechanism with fixed-rate epoch rewards (~1.92M WP per epoch), 2-epoch unstake delay
- **Execution-fee reserve** funded by a 1% revenue cut plus stake/unstake fees (self-sustainability)
- **Shareholder governance:** change the admin / reinvest address by >51% of the 676 SC shares

**Intended Benefits:**
- Trustless, verifiable distribution eliminating intermediaries
- Self-sustaining operation: the contract funds its own execution fees instead of relying on a one-time reserve
- Decentralized control: no privileged admin key — admin/reinvest changes require a shareholder supermajority
- State-bloat resistant: dust thresholds and minimum-stake rules keep the recurring workload bounded

---

## Technical Description

**Contract Metadata:**
- **Language:** Restricted C++17 (Qubic Protocol Interface)
- **Contract Index:** 28
- **Construction Epoch:** 218 (Proposal: Epoch 216, IPO: Epoch 217)
- **Distribution cadence:** once per epoch (weekly) in `END_EPOCH` — no time/day gate
- **Max Supported Token Holders:** 16,384 (per-epoch snapshot, dust-filtered)

**External Token:**
- **Name:** WP (WolfPack token)
- **Issuer:** MLMWPSQNVAIBRFDHWCKSFOVUAZDDWKJGCLRSYZIUEFDURPWIPQXACYOE
- **Type:** External asset issued on QX (Qubic exchange layer)
- **Role in GGWP:** Primary revenue distribution asset; staking collateral

**Asset Encoding:**
- GGWP SC asset name: 1347897159 ("GGWP" in ASCII) — the 676 IPO shares (issuer = NULL_ID)
- SC shareholders are paid via `qpi.distributeDividends()`; voting power is read live via `qpi.numberOfShares()`

---

## Functional Specifications

### Revenue Distribution (weekly, END_EPOCH)

Accumulated `pendingRevenue` is split once per epoch:

| Portion | Permille | Recipient |
|---------|----------|-----------|
| Holders | 700 (70%) | WP token holders, proportional to the BEGIN_EPOCH snapshot |
| Shareholders | 100 (10%) | 676 SC shares via `qpi.distributeDividends()` |
| Clan | 100 (10%) | active clan members, rank-weighted |
| Reinvest | 90 (9%) | reinvestment address (governable) |
| Exec reserve | 10 (1%) | retained in-contract for execution fees |

The five portions sum to exactly 1000‰ (compile-time checked via `static_assert`).

### Clan Rank System

**6 Membership Tiers with Reward Multipliers:**
| Rank | Title | Multiplier |
|------|-------|------------|
| 0 | Recruit | 1.0x |
| 1 | Private | 1.3x |
| 2 | Sergeant | 1.8x |
| 3 | Lieutenant | 2.5x |
| 4 | Colonel | 3.2x |
| 5 | General | 4.0x |

### Staking Mechanism

**Deposit Phase:**
1. User calls `QX.TransferShareManagementRights` to delegate WP to GGWP
2. User calls `Stake(amount)` → shares locked under contract management
3. **Minimum stake: 500,000 WP** (resulting position must reach the minimum)
4. **Stake fee: 900 QU** retained in the execution-fee reserve (causer-pays)

**Reward Distribution (BEGIN_EPOCH):**
- Fixed pool: `WOLFPACK_STAKING_REWARD_PER_EPOCH = 1,923,076` (~100M ÷ 52 epochs)
- Proportional: `reward[user] = (rewardPool ÷ totalStaked) × userStaked` (uint128 intermediates)

**Withdrawal Phase:**
1. `RequestUnstake(amount)` → remaining position must be 0 (full exit) or ≥ 500,000
2. After a 2-epoch delay: `FinalizeUnstake()` releases shares via `QX.ReleaseShares`
3. **Unstake fee: 1,000 QU** = 100 (QX release fee) + 900 (execution-fee reserve)

### Shareholder Governance

Admin and reinvest addresses are changed **only** by a vote of the SC shareholders (676 shares = 100%).

- `ProposeGovChange(targetType, newAddress)` — any shareholder opens a proposal (admin or reinvest)
- `VoteGovChange(proposalIndex, approve)` — one vote per shareholder, weighted by live share count
- **Threshold:** ≥ 51% of 676 shares (≥ 345 shares) executes the change immediately
- **Up to 8 concurrent proposals**, each with a unique id (no slot-recycling double-counting)
- Voting power is read live via `qpi.numberOfShares()`; transfers cannot double-count
- `NULL_ID` targets are rejected

### Administration

`SetAdmin` is **deprecated** (always rejects) to prevent NULL_ID seizure. Clan/exclude configuration remains admin-only, but the admin address itself is replaceable only via the >51% shareholder vote.

**Read Functions:** `GetStatus`, `GetHolderInfo`, `GetShareholderInfo`, `GetClanMemberInfo`, `GetStakingInfo`, `GetDistributionPreview`, `GetGovProposal`.

---

## Security Properties

| Property | Mechanism |
|----------|-----------|
| **No flash-loan dividend capture** | BEGIN_EPOCH snapshot for holders |
| **No double-payout** | distribution runs once per epoch, gated on `pendingRevenue` |
| **No admin drain** | admin cannot move funds; admin/reinvest only via >51% vote |
| **No overflow** | uint128 intermediates for all share math |
| **No unbounded loops** | iteration bounded by MAX_HOLDERS; dust threshold limits the holder set |
| **No vote double-count** | live share tally + unique proposal ids |
| **Self-sustaining** | execution-fee reserve fed by 1% cut + stake/unstake fees |
| **State-bloat resistant** | dust threshold (10k) + minimum stake (500k) |
| **Immutable admin key** | SetAdmin deprecated; governance vote is the only path |

---

## Technical Implementation

### Pull Request & Code

- **PR:** [https://github.com/qubic/core/pull/912/changes](https://github.com/qubic/core/pull/912/changes) - GGWP Smart Contract (revenue distribution, staking, governance)
- **Branch:** `ggwp-develop` (based on `qubic/core:develop`)

### Source Code Files

- **Contract:** [`src/contracts/GGWP.h`](https://github.com/MZoxx/core/blob/ggwp-develop/src/contracts/GGWP.h)
  - Weekly revenue distribution (5 portions) in `END_EPOCH`
  - Staking with fixed-rate rewards, stake/unstake fees → execution-fee reserve
  - Shareholder governance (multi-proposal, live voting power)
  - Clan management with rank-weighted multipliers
- **Registration:** [`src/contract_core/contract_def.h`](https://github.com/MZoxx/core/blob/ggwp-develop/src/contract_core/contract_def.h) — index 28, construction epoch 218
- **Tests:** [`test/contract_ggwp.cpp`](https://github.com/MZoxx/core/blob/ggwp-develop/test/contract_ggwp.cpp) — 51 unit tests

### Test Coverage

**51 Unit Tests** covering:
- Revenue split math incl. the execution-fee reserve
- Staking deposit/withdrawal, reward distribution, overflow edge case
- Minimum stake and unstake-consistency (0 or ≥ 500k)
- Shareholder governance (propose, vote, threshold, multiple proposals, invalid index)
- Clan management, admin access control, lifecycle (BEGIN_EPOCH/END_EPOCH)

**All tests passing** on Linux x86-64 (clang); contract passes the Qubic contract-verification tool.

### Constants & Parameters

```cpp
// Distribution (permille; sum == 1000)
constexpr uint64 WOLFPACK_DISTRIBUTION_PERMILLE_HOLDERS      = 700; // 70%
constexpr uint64 WOLFPACK_DISTRIBUTION_PERMILLE_SHAREHOLDERS = 100; // 10%
constexpr uint64 WOLFPACK_DISTRIBUTION_PERMILLE_CLAN         = 100; // 10%
constexpr uint64 WOLFPACK_DISTRIBUTION_PERMILLE_REINVEST     = 90;  //  9%
constexpr uint64 WOLFPACK_DISTRIBUTION_PERMILLE_EXEC_RESERVE = 10;  //  1%

// Staking / governance / anti-dust
constexpr uint64 WOLFPACK_STAKING_REWARD_PER_EPOCH = 1923076ULL;    // ~100M / 52
constexpr sint64 WOLFPACK_STAKE_FEE       = 900LL;   // QU -> execution-fee reserve
constexpr sint64 WOLFPACK_QX_TRANSFER_FEE = 100LL;   // QX release fee (paid by user)
constexpr uint64 WOLFPACK_MIN_STAKE              = 500000;  // min staked position
constexpr uint64 WOLFPACK_MIN_ELIGIBLE_BALANCE   = 10000;   // holder dust threshold
constexpr uint64 WOLFPACK_TOTAL_SC_SHARES        = 676;     // IPO shares = 100%
constexpr uint64 WOLFPACK_GOV_THRESHOLD_PERCENT  = 51;      // >= 51% to execute
constexpr uint64 WOLFPACK_MAX_GOV_PROPOSALS      = 8;       // concurrent proposals
```

---

## Deployment Details

**Deployment Process:**
1. **Epoch 216:** GQMPROP voting on this proposal
2. **Epoch 217:** If approved, code merged; IPO conducted; QUs collected and burned
3. **Epoch 218:** Contract constructed; INITIALIZE executes; ready for user calls

**IPO Mechanism:**
- Shareholders purchase SC shares via GQMPROP automatic price discovery
- IPO shares: 676 total (proportional distribution)
- Collected QUs: Burned to seed the execution-fee reserve (thereafter topped up by the 1% cut + stake/unstake fees)
- Admin role: Initialized to the token issuer; thereafter changeable only by >51% shareholder vote

**Mainnet Readiness:**
- ✓ 51/51 unit tests passing
- ✓ Contract verifies (Qubic contract-verification tool compliance)
- ✓ Builds against `qubic/core:develop` QPI
- ✓ Self-sustainability, anti-dust and governance hardening reviewed

---

## Conclusion

GGWP brings trustless, deterministic, **self-sustaining** revenue distribution to the Qubic network, with staking and on-chain shareholder governance. By funding its own execution fees and removing the privileged admin key in favor of a >51% shareholder vote, it is designed for long-term, operator-neutral operation.

Approval enables WP token holders to begin receiving proportional revenue distributions and staking rewards, while shareholders retain decentralized control over the contract's configurable addresses.

**Recommended for approval.**
