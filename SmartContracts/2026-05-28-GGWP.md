# Smart Contract - GGWP (WolfPack)

## Proposal Statement

Allow deployment of the GGWP (WolfPack) smart contract on the Qubic network, enabling automated revenue distribution and staking mechanisms for WP token holders, shareholders, and clan members.

**Available Options:**
- Option 0: No - Do not deploy GGWP
- Option 1: Yes - Deploy GGWP with revenue distribution and staking

---

## Executive Summary

GGWP (WolfPack) is a non-custodial, deterministic smart contract that programmatically distributes recurring revenue from multiple sources directly to on-chain stakeholders without oracles or admin control over payouts.

**Key Capabilities:**
- Automated daily revenue distribution (11:00 UTC) with 70/10/10/10 split
- Token holder distribution (WP token snapshots at epoch boundaries)
- Shareholder dividends (676 IPO shares from contract initialization)
- Clan member rewards (weighted by 6 rank tiers: 1.0x to 4.0x multipliers)
- Reinvestment fund (hardcoded destination for ecosystem growth)
- Staking mechanism with fixed-rate epoch rewards (~1.92M WP per epoch)
- 2-epoch unstake delay preventing flash-loan attacks

**Intended Benefits:**
- Trustless, verifiable distribution eliminating intermediaries
- Incentivizes long-term participation through rank-weighted clan rewards
- Separates operational equity (token holders) from capital equity (shareholders)
- Enables immediate staking participation without complex governance

---

## Technical Description

**Contract Metadata:**
- **Language:** Restricted C++17 (Qubic Protocol Interface)
- **Contract Index:** 28
- **Construction Epoch:** 217 (Proposal: Epoch 215, IPO: Epoch 216)
- **State Size:** ~10 KB (per holder/shareholder)
- **Max Supported Holders:** 131,072 unique addresses
- **Max Held Assets:** 1,024 distinct asset types

**External Token:**
- **Name:** WP (WolfPack token)
- **Issuer:** MLMWPSQNVAIBRFDHWCKSFOVUAZDDWKJGCLRSYZIUEFDURPWIPQXACYOE
- **Type:** External asset issued on QX (Qubic exchange layer)
- **Role in GGWP:** Primary revenue distribution asset; staking collateral

**Asset Encoding:**
- GGWP smart contract asset name: 1347897159 (0x50574747 = "GGWP" in ASCII)
- Used internally for tracking SC shareholdings

---

## Functional Specifications


### Clan Rank System

**6 Membership Tiers with Reward Multipliers:**
| Rank | Title | Multiplier | Use Case |
|------|-------|------------|----------|
| 0 | Recruit | 1.0x | Entry level |
| 1 | Private | 1.3x | Active member |
| 2 | Sergeant | 1.8x | Leadership |
| 3 | Lieutenant | 2.5x | Senior leadership |
| 4 | Colonel | 3.2x | Strategic roles |
| 5 | General | 4.0x | Founders/Core |

**Weighted Calculation:**
```
clanWeightedTotal = sum(memberCount × rankMultiplier × 1000)
clanReward[member] = (memberRank × 1000) × qrwaPortion / clanWeightedTotal
```

### Staking Mechanism

**Deposit Phase:**
1. User calls `QX.TransferShareManagementRights` to delegate WP to GGWP
2. User calls `Stake(amount)` → shares locked under contract management
3. Staked amount tracked in per-user HashMap

**Reward Distribution (BEGIN_EPOCH):**
- Fixed pool: `WOLFPACK_STAKING_REWARD_PER_EPOCH = 1,923,076 QU` (~100M ÷ 52 epochs)
- Proportional: `reward[user] = (rewardPool ÷ totalStaked) × userStaked`
- Remainder handling: Uses uint128 arithmetic for precision
- Accumulates in `pendingStakingRewards` HashMap

**Withdrawal Phase:**
1. User calls `RequestUnstake(amount)` → marked pending at current epoch
2. After 2-epoch delay: User calls `FinalizeUnstake()`
3. Calls `QX.ReleaseShares(amount)` → shares return to user control
4. QX fee: 100 QU per release (paid by user)

### Governance & Administration

**Immutable Admin Address:**
- Hardcoded at INITIALIZE to token issuer: `MLMWPSQNVAIBRFDHWCKSFOVUAZDDWKJGCLRSYZIUEFDURPWIPQXACYOE`
- SetAdmin deprecated (always rejects) to prevent NULL_ID attacks
- Admin cannot move funds; can only configure addresses and manage clan

**Admin Capabilities:**
- `AddClanMember(address, rank)` - Add member to clan
- `RemoveClanMember(address)` - Remove member
- `SetClanRank(address, newRank)` - Update rank tier
- `SetExcludeAddress(slot, address)` - Configure 2 exclude addresses (exchanges)
- `DepositStakingRewards(amount)` - Fund reward pool

**Read Functions:**
- `GetStatus()` - Contract state snapshot
- `GetHolderInfo(address)` - Holder balance & distribution info
- `GetShareholderInfo(address)` - Shareholder balance
- `GetClanMemberInfo(address)` - Clan rank & rewards
- `GetStakingInfo(address)` - Stake amount, pending rewards
- `GetDistributionPreview()` - Next payout estimate

---

## Security Properties

| Property | Mechanism |
|----------|-----------|
| **No flash-loan dividend capture** | Snapshot-based (min(beginBalance, endBalance)) |
| **No double-payout** | 6-day per-stream cooldown gated on actual revenue |
| **No admin drain** | Admin hardcoded, cannot move treasury funds |
| **No overflow** | uint128 intermediates for all share math |
| **No unbounded loops** | All iteration bounded by MAX_HOLDERS / MAX_ASSETS |
| **No unintended recursion** | SELF filtered from all iterator passes |
| **Immutable admin** | SetAdmin deprecated to prevent NULL_ID attacks |

---

## Technical Implementation

### Pull Request & Code

The complete implementation is available in:
- **PR:** [https://github.com/qubic/core/pull/880/changes](https://github.com/qubic/core/pull/880/changes) - GGWP Smart Contract with Revenue Distribution & Staking
- **Branch:** `feature/2026-05-18-ggwp-contract`

### Source Code Files

Full contract implementation:
- **Contract:** [`src/contracts/GGWP.h`](https://github.com/MZoxx/core/blob/feature/2026-05-18-ggwp-contract/src/contracts/GGWP.h) (967 lines)
  - Revenue distribution logic (4 independent streams)
  - Staking mechanism with fixed-rate rewards
  - Clan member management with rank-weighted multipliers
  - Admin procedures for configuration and governance
  
- **Registration:** [`src/contract_core/contract_def.h`](https://github.com/MZoxx/core/blob/feature/2026-05-18-ggwp-contract/src/contract_core/contract_def.h#L411) (line 411)
  - Contract index: 28
  - Construction epoch: 217

- **Tests:** [`test/contract_ggwp.cpp`](https://github.com/MZoxx/core/blob/feature/2026-05-18-ggwp-contract/test/contract_ggwp.cpp)
  - 39 comprehensive unit tests

### Test Coverage

**39 Unit Tests** covering:
- Revenue distribution logic (splits, rounding, remainder handling)
- Staking deposit, withdrawal, reward distribution
- Clan member management (add, remove, rank updates)
- Admin controls and access restrictions
- Edge cases (zero amounts, insufficient stake, pending unstake blocking)
- State consistency after BEGIN_EPOCH/END_EPOCH lifecycle

**All tests passing** on:
- Linux (GCC/Clang, x86-64 and ARM64)
- Windows (MSVC, x64 EFI build)

### Constants & Parameters

```cpp
// Revenue
constexpr uint64 WOLFPACK_STAKING_REWARD_PER_EPOCH = 1923076ULL;  // ~100M / 52 weeks
constexpr uint64 PERMILLE_HOLDERS = 700;      // 70% to token holders
constexpr uint64 PERMILLE_SHAREHOLDERS = 100; // 10% to shareholders
constexpr uint64 PERMILLE_CLAN = 100;         // 10% to clan members
constexpr uint64 PERMILLE_REINVEST = 100;     // 10% to reinvestment

// Clan ranks (multipliers in permille)
constexpr uint64 CLAN_RANK_MULTIPLIERS[6] = {1000, 1300, 1800, 2500, 3200, 4000};

// Limits
constexpr uint64 WOLFPACK_MAX_HOLDERS = 131072;
constexpr uint64 WOLFPACK_MAX_ASSETS = 1024;
constexpr uint64 WOLFPACK_MAX_CLAN_MEMBERS = 8192;
```

---

## Deployment Details

**Deployment Process:**
1. **Epoch N (current):** GQMPROP voting on this proposal
2. **Epoch N+1:** If approved, code merged; IPO conducted; QUs collected and burned
3. **Epoch N+2 (217):** Contract constructed; INITIALIZE executes; ready for user calls

**IPO Mechanism:**
- Shareholders purchase SC shares via GQMPROP automatic price discovery
- IPO shares: 676 total (proportional distribution)
- Collected QUs: Burned to fund execution fee reserve
- Admin role: Automatically transferred to token issuer

**Mainnet Readiness:**
- ✓ 39/39 unit tests passing
- ✓ Contract verifies (Qubic verification tool compliance)
- ✓ Windows EFI build successful
- ✓ Linux/ARM64 builds successful
- ✓ Security audit: Immutable admin, uint128 overflow prevention, snapshot fairness

---

## Conclusion

GGWP brings trustless, deterministic revenue distribution to the Qubic network while maintaining full on-chain auditability and operator neutrality. The contract serves as a reference implementation for dual-mechanism contracts (revenue distribution + staking) and establishes patterns for future community-driven SC initiatives.

Approval enables WP token holders to immediately begin receiving proportional revenue distributions and staking rewards, aligning incentives across multiple stakeholder classes without intermediary risk.

**Recommended for approval.**
