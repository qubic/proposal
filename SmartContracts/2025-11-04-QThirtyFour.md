# QThirtyFour (QTF) Technical Specification

## Contract Overview

**Contract Name:** QThirtyFour (QTF)
**Type:** Lottery smart contract
**Platform:** Qubic blockchain
**Version:** 1.0
**Status:** Development/Testing

## Available Options
> Option 0: no, dont allow

> Option 1: yes, allow

## Table of Contents

1. [Introduction](#1-introduction)
2. [Game Mechanics](#2-game-mechanics)
3. [Revenue Distribution](#3-revenue-distribution)
4. [Prize Pool Structure](#4-prize-pool-structure)
5. [Fast-Recovery System](#5-fast-recovery-system)
6. [Reserve Pool Integration](#6-reserve-pool-integration)
7. [Settlement Process](#7-settlement-process)
8. [Security Features](#8-security-features)
9. [Configuration Parameters](#9-configuration-parameters)
10. [Code](#10-code)
11. [Glossary](#glossary)

---

## 1. Introduction

QThirtyFour is a 4-of-30 lottery smart contract that operates on the Qubic blockchain. Players select 4 unique numbers from a range of 1-30, and prizes are awarded based on how many numbers match the randomly drawn winning numbers.

### Key Features

- **Three prize tiers:** k=2, k=3, and k=4 (jackpot)
- **Guaranteed minimum payouts** with reserve pool backing
- **Adaptive Fast-Recovery (FR) system** for jackpot rebuilding
- **Scheduled draws** with configurable draw days and times
- **Transparent prize pool calculations** visible before draws
- **Integration with QReservePool (QRP)** for financial stability

---

## 2. Game Mechanics

### Number Selection

- Players choose **4 unique numbers** from the range **[1, 30]**
- All numbers must be unique within a single ticket
- Invalid selections are rejected with appropriate error codes

### Ticket Purchase

1. Player invokes `BuyTicket` procedure with:
   - Payment: Ticket price in QU (default: 1,000,000 QU)
   - Selection: Array of 4 unique numbers [1-30]

2. Contract validates:
   - Ticket price is correct (exact payment or overpayment with refund)
   - Numbers are valid and unique
   - Player limit not exceeded (max 1,024 players per round)
   - Selling is currently active

3. If valid: Ticket is registered, excess payment refunded
4. If invalid: Full refund with error code

### Winning Criteria

- **k=2 (Second Tier):** Match exactly 2 of 4 numbers
- **k=3 (Third Tier):** Match exactly 3 of 4 numbers
- **k=4 (Jackpot):** Match all 4 numbers

### Draw Schedule

- **Primary draw:** Every Wednesday at configured hour (default: 11:00 UTC)
- **Additional draws:** Configurable via schedule bitmask (7 days of week)
- **Draw frequency:** Throttled to once per day maximum
- **Settlement:** Triggered automatically by BEGIN_TICK system procedure

---

## 3. Revenue Distribution

### Default Fee Structure (from RandomLottery contract)

When tickets are sold, revenue is split according to fees:

| Recipient | Percentage | Description |
|-----------|------------|-------------|
| Winners Block | 68% | Allocated to prize pools (k2, k3, jackpot) |
| Distribution (Shareholders) | 20% | RL asset holders (computors) |
| Development Team | 10% | Contract development and maintenance |
| Burn | 2% | Deflationary mechanism |
| **Total** | **100%** | |

**Example:** 100 tickets × 1M QU = 100M QU revenue
- Winners: 68M QU
- Distribution: 20M QU
- Dev: 10M QU
- Burn: 2M QU

---

## 4. Prize Pool Structure

### Baseline Mode (FR Inactive)

When Fast-Recovery is **OFF**, the Winners Block (68% of revenue) is split:

| Pool | Percentage of Winners Block | Example (68M QU) |
|------|----------------------------|------------------|
| k=3 Pool | 40% | 27.2M QU |
| k=2 Pool | 28% | 19.04M QU |
| Overflow | 32% | 21.76M QU |

**Overflow Split (50/50):**
- 50% → Reserve Pool (QRP)
- 50% → Jackpot carry

### Fast-Recovery Mode (FR Active)

When FR is **ON**, additional mechanisms activate:

#### 1. Winners Rake (5%)
- **5% of Winners Block** redirected to jackpot
- Example: 68M × 5% = 3.4M QU to jackpot
- Remaining 95% split into k2/k3 pools

#### 2. Base Redirects (Always Applied in FR)
- **Dev redirect:** 1.00% of revenue → Jackpot
- **Dist redirect:** 1.00% of revenue → Jackpot
- Deducted from Dev and Distribution payouts

#### 3. Extra Redirects (Deficit-Driven)
Calculated dynamically based on:
- **Deficit:** Δ = max(0, targetJackpot - currentJackpot)
- **Expected rounds to k=4:** E_k4(N) = 1 / (1 - (1-p4)^N)
  - p4 = 1/27,405 (combinatorial probability)
  - N = number of tickets sold
- **Horizon:** H = min(E_k4(N), 50 rounds)
- **Needed gain:** g_need = max(0, Δ/H - baseGain)
- **Extra percentage:** extra_pp = clamp(g_need / R, 0, 0.70%)

Split equally between Dev and Dist (max 0.35% each).

#### 4. Overflow Bias (95/5)
- **95% of overflow** → Jackpot
- **5% of overflow** → Reserve Pool

**FR Mode Winners Block Split:**

| Component | Calculation |
|-----------|-------------|
| Winners Block | 68% of revenue |
| Winners Rake (5%) | → Jackpot |
| Effective Winners Block | 95% of original |
| k=3 Pool | 40% of effective |
| k=2 Pool | 28% of effective |
| Overflow | 32% of effective |

---

## 5. Fast-Recovery System

### Purpose

Rebuild jackpot to target level after a k=4 win as quickly as possible while maintaining player value.

### Activation Logic

FR activates when **BOTH** conditions are true:
1. **Jackpot < Target Jackpot**
2. **Rounds since last k=4 < 50** (post-k4 window)

### Deactivation Logic (Hysteresis)

FR deactivates when:
- **Jackpot ≥ Target Jackpot** for **3 consecutive rounds**

This prevents rapid on/off toggling near the threshold.

### FR Mechanisms Summary

| Mechanism | Rate | Destination |
|-----------|------|-------------|
| Winners Rake | 5% of Winners Block | Jackpot |
| Base Dev Redirect | 1% of Revenue | Jackpot |
| Base Dist Redirect | 1% of Revenue | Jackpot |
| Extra Dev Redirect | 0-0.35% of Revenue | Jackpot |
| Extra Dist Redirect | 0-0.35% of Revenue | Jackpot |
| Overflow Bias | 95% of Overflow | Jackpot |

### Post-K4 Reseed

When jackpot is won (k=4):
1. Jackpot paid out to winners (divided equally if multiple)
2. Jackpot balance reset to 0
3. **Reseed from QRP:** Up to min(QRP balance, targetJackpot)
4. FR counters reset
5. FR activates for next round

---

## 6. Reserve Pool Integration

QThirtyFour integrates with **QReservePool (QRP)** contract for financial stability.

### QRP Functions Used

| Function | Purpose |
|----------|---------|
| `GetAvailableReserve` | Query total reserve available |
| `GetReserve` | Request funds from reserve |

### Reserve Usage Scenarios

#### 1. Floor Guarantee Top-Up

If prize pool insufficient to meet minimum floors:
- **k=2 floor:** 0.5 × Ticket Price per winner
- **k=3 floor:** 5 × Ticket Price per winner

**Safety Limits:**
- Maximum 10% of QRP balance per round
- Soft floor: Keep ≥20 × P in QRP
- Per-winner cap: 25 × P maximum

#### 2. Jackpot Reseed (Post-K4)

After k=4 win:
- Request: min(QRP balance, targetJackpot)
- No safety limits (jackpot critical)

#### 3. Overflow Deposit

Unawarded funds sent to QRP:
- **Baseline mode:** 50% of overflow → QRP
- **FR mode:** 5% of overflow → QRP

---

## 7. Settlement Process

### Trigger

Settlement occurs automatically via `BEGIN_TICK` system procedure when:
1. Current hour ≥ configured draw hour
2. Scheduled day (Wednesday or schedule bit set)
3. Different date from last draw

### SettleEpoch Detailed Steps

#### Phase 1: Validation
1. Check if players > 0 (else return)
2. Calculate revenue (ticketPrice × numberOfPlayers)
3. Verify contract balance ≥ revenue (else refund all)

#### Phase 2: Fee Calculation
4. Query fees from RL contract
5. Calculate base payouts:
   - devPayout = revenue × 10%
   - distPayout = revenue × 20%
   - burnAmount = revenue × 2%

#### Phase 3: Prize Pool Calculation
6. Call `CalculatePrizePools`:
   - Calculate winnersBlock (68% of revenue)
   - Apply FR rake if active (5% to jackpot)
   - Split into k2 (28%) and k3 (40%) pools
   - Calculate overflow (32% remaining)

#### Phase 4: FR Hysteresis & Activation
7. Update hysteresis counter:
   - If jackpot ≥ target: increment frRoundsAtOrAboveTarget
   - Else: reset to 0
8. Determine FR activation:
   - Activate if: jackpot < target AND roundsSinceK4 < 50
   - Deactivate if: frRoundsAtOrAboveTarget ≥ 3

#### Phase 5: FR Redirects (if active)
9. Calculate deficit: Δ = max(0, target - jackpot)
10. Estimate base gain (1% Dev + 1% Dist + 5% rake + 95% overflow)
11. Calculate extra redirect based on deficit/horizon
12. Compute total redirects (base + extra)
13. Deduct from Dev/Dist payouts (capped at available)

#### Phase 6: Random Number Generation
14. Generate 4 unique winning numbers [1-30] using K12 hash

#### Phase 7: Winner Counting
15. First pass through players:
    - Count matches for each ticket (k=2, k=3, k=4)
    - Cache results for second pass

#### Phase 8: Tier Payouts
16. Query QRP available reserve
17. Process k=2 tier:
    - Call `ProcessTierPayout` with floor 0.5×P
    - Top-up from QRP if needed
    - Calculate per-winner payout (capped at 25×P)
    - Collect overflow
18. Process k=3 tier:
    - Call `ProcessTierPayout` with floor 5×P
    - Top-up from QRP if needed
    - Calculate per-winner payout (capped at 25×P)
    - Collect overflow

#### Phase 9: Winner Payments
19. Second pass through players:
    - Pay k=2 winners (if countK2 > 0)
    - Pay k=3 winners (if countK3 > 0)
    - Pay k=4 winners (jackpot / countK4)
    - Record winners in lastWinnerData

#### Phase 10: Jackpot Handling
20. If k=4 won:
    - Deplete jackpot to 0
    - Reset FR counters
    - Reseed from QRP (up to min(QRP, target))
21. Else:
    - Increment frRoundsSinceK4

#### Phase 11: Overflow Distribution
22. Split overflow:
    - FR mode: 95% → jackpot, 5% → QRP
    - Baseline: 50% → jackpot, 50% → QRP

#### Phase 12: Jackpot Update
23. Add to jackpot:
    - Winners rake (if FR)
    - Dev redirects (if FR)
    - Dist redirects (if FR)
    - Overflow carry portion

#### Phase 13: External Transfers
24. Transfer overflow reserve to QRP
25. Transfer dev payout to teamAddress
26. Distribute dist payout to RL shareholders:
    - Iterate through RL asset holders
    - Calculate per-share dividend
    - Transfer to each holder
    - Return remainder to RL contract
27. Burn burnAmount

---

## 8. Security Features

### Input Validation

1. **Number validation:**
   - Range check: 1 ≤ value ≤ 30
   - Uniqueness check via HashSet
   - ASSERT on array access to prevent overflow

2. **Payment validation:**
   - Exact or higher payment accepted
   - Excess refunded immediately
   - Zero-payment rejected

3. **Access control:**
   - Owner-only procedures (SetPrice, SetSchedule, etc.)
   - Non-owner invocations rejected with ACCESS_DENIED

### Economic Safety

1. **Balance verification:**
   - Check contract balance ≥ revenue before settlement
   - Refund all players if insufficient

2. **Reserve limits:**
   - Max 10% QRP per round for top-ups
   - Soft floor: Keep ≥20×P in QRP
   - Per-winner cap: 25×P maximum

3. **Overflow protection:**
   - Safe arithmetic: `sadd()`, `smul()`, `div()`
   - No raw `+` or `*` in financial calculations
   - Division by zero prevented

### State Integrity

1. **Ticket selling gates:**
   - Disabled during settlement
   - Disabled on Wednesdays (main draw day)
   - Max 1,024 players enforced

2. **Draw scheduling:**
   - Date stamp prevents same-day re-draws
   - Throttled to RL_TICK_UPDATE_PERIOD

3. **FR counters:**
   - Reset on jackpot win
   - Hysteresis prevents rapid toggling

---

## 9. Configuration Parameters

### Core Game Parameters

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_MAX_NUMBER_OF_PLAYERS` | 1024 | Max tickets per round |
| `QTF_RANDOM_VALUES_COUNT` | 4 | Numbers per ticket |
| `QTF_MAX_RANDOM_VALUE` | 30 | Number range [1-30] |
| `QTF_TICKET_PRICE` | 1,000,000 QU | Default ticket price |

### Prize Pool Splits (Baseline)

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_BASE_K3_SHARE_BP` | 4000 | k=3 gets 40% of winners block |
| `QTF_BASE_K2_SHARE_BP` | 2800 | k=2 gets 28% of winners block |
| (Overflow) | 3200 | 32% remaining goes to overflow |

### Fast-Recovery Parameters

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_FR_DEV_REDIRECT_BP` | 100 | 1.00% base dev redirect |
| `QTF_FR_DIST_REDIRECT_BP` | 100 | 1.00% base dist redirect |
| `QTF_FR_EXTRA_MAX_BP` | 70 | 0.70% max extra redirect total |
| `QTF_FR_WINNERS_RAKE_BP` | 500 | 5% of winners block |
| `QTF_FR_ALPHA_BP` | 500 | 5% overflow to reserve (95% to jackpot) |
| `QTF_FR_POST_K4_WINDOW_ROUNDS` | 50 | Rounds after k=4 to keep FR eligible |
| `QTF_FR_HYSTERESIS_ROUNDS` | 3 | Rounds at target before FR off |
| `QTF_FR_GOAL_ROUNDS_CAP` | 50 | Max horizon for deficit calculation |
| `QTF_P4_DENOMINATOR` | 27405 | Exact k=4 probability: 1/27405 |

### Payout Floors and Caps

| Constant | Value | Calculation | Example (P=1M) |
|----------|-------|-------------|----------------|
| `QTF_K2_FLOOR_MULT` | 1 | 1×P / 2 | 500k QU |
| `QTF_K2_FLOOR_DIV` | 2 | | |
| `QTF_K3_FLOOR_MULT` | 5 | 5×P | 5M QU |
| `QTF_TOPUP_PER_WINNER_CAP_MULT` | 25 | 25×P | 25M QU |

### Reserve Safety Limits

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_TOPUP_RESERVE_PCT_BP` | 1000 | Max 10% QRP per round |
| `QTF_RESERVE_SOFT_FLOOR_MULT` | 20 | Keep ≥20×P in QRP |

### Overflow Split (Baseline)

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_BASELINE_OVERFLOW_ALPHA_BP` | 5000 | 50% reserve, 50% jackpot |

### Defaults

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_DEFAULT_TARGET_JACKPOT` | 1,000,000,000 QU | 1 billion QU |
| `QTF_DEFAULT_SCHEDULE` | 0x08 | Wednesday only |
| `QTF_DEFAULT_DRAW_HOUR` | 11 | 11:00 UTC |

### Default Fees (Fallback)

| Constant | Value | Description |
|----------|-------|-------------|
| `QTF_DEFAULT_DEV_PERCENT` | 10 | 10% to dev |
| `QTF_DEFAULT_DIST_PERCENT` | 20 | 20% to shareholders |
| `QTF_DEFAULT_BURN_PERCENT` | 2 | 2% burned |
| `QTF_DEFAULT_WINNERS_PERCENT` | 68 | 68% to winners |

---

## Appendix A: Probability Calculations

### K=4 Win Probability

**Combinatorics:**
- Total combinations: C(30,4) = 27,405
- Winning combinations: C(4,4) × C(26,0) = 1
- **Probability: p4 = 1 / 27,405 ≈ 0.00365%**

### Expected Rounds to K=4

Given N tickets per round:

**Formula:**
```
E_k4(N) = 1 / (1 - (1 - p4)^N)
```

**Examples:**
| Tickets (N) | Approx E_k4 |
|-------------|-------------|
| 100 | ~274 rounds |
| 500 | ~55 rounds |
| 1000 | ~27 rounds |

This drives the FR extra redirect calculation.

---

## Appendix C: Example Scenarios

### Scenario 1: Baseline Round (FR Off)

**Setup:**
- Tickets sold: 200
- Ticket price: 1M QU
- FR: Inactive
- Jackpot: 1.5B QU (above target)

**Revenue Distribution:**
- Total: 200M QU
- Dev: 20M (10%)
- Dist: 40M (20%)
- Burn: 4M (2%)
- Winners: 136M (68%)

**Prize Pools:**
- k=3: 54.4M (40% of 136M)
- k=2: 38.08M (28% of 136M)
- Overflow: 43.52M (32% of 136M)

**Overflow Split (50/50):**
- To QRP: 21.76M
- To Jackpot: 21.76M

**Winners (example):**
- k=2: 15 winners → 2.54M each (38.08M / 15)
- k=3: 3 winners → 18.13M each (54.4M / 3)
- k=4: 0 winners → Jackpot carries

**Final State:**
- Jackpot: 1.52176B (+21.76M)
- QRP: +21.76M
- Rounds since k4: +1

---

### Scenario 2: FR Active Round

**Setup:**
- Tickets sold: 500
- Ticket price: 1M QU
- FR: Active
- Jackpot: 500M QU (below 1B target)
- Deficit: 500M QU

**Revenue Distribution:**
- Total: 500M QU
- Base Dev: 50M (10%)
- Base Dist: 100M (20%)
- Burn: 10M (2%)
- Winners: 340M (68%)

**Prize Pools (FR):**
- Winners rake (5%): 17M → Jackpot
- Effective winners: 323M
- k=3: 129.2M (40% of 323M)
- k=2: 90.44M (28% of 323M)
- Overflow: 103.36M (32% of 323M)

**FR Redirects:**
- Base dev redirect (1%): 5M
- Base dist redirect (1%): 5M
- Extra redirect (calculated): ~2M total (1M dev, 1M dist)
- Total redirects: 12M (5+5+1+1)

**Adjusted Payouts:**
- Dev: 50M - 6M = 44M
- Dist: 100M - 6M = 94M

**Overflow Split (95/5):**
- To Jackpot: 98.19M (95% of 103.36M)
- To QRP: 5.17M (5% of 103.36M)

**Total to Jackpot:**
- Winners rake: 17M
- Redirects: 12M
- Overflow: 98.19M
- **Total: 127.19M**

**Winners (example):**
- k=2: 40 winners → 2.26M each (90.44M / 40)
- k=3: 5 winners → 25M each (capped at 25M, was 25.84M)
- k=4: 0 winners

**Final State:**
- Jackpot: 627.19M (+127.19M)
- QRP: +5.17M
- Rounds since k4: +1
- FR stays active (still below target)

---

### Scenario 3: Jackpot Win

**Setup:**
- Tickets sold: 800
- Ticket price: 1M QU
- FR: Active
- Jackpot: 1.2B QU

**Revenue:**
- Total: 800M QU

**Winners:**
- k=2: 50 winners
- k=3: 8 winners
- **k=4: 2 winners** ← Jackpot won!

**Jackpot Payout:**
- Per winner: 1.2B / 2 = 600M QU each
- Total: 1.2B depleted

**Post-Win:**
1. Jackpot reset to 0
2. FR counters reset
3. Reseed from QRP:
   - Request: min(QRP balance, 1B target)
   - Assume QRP has 2B available
   - **Receive: 1B QU**
4. New jackpot: 1B QU
5. FR activates (jackpot = target, but will deactivate after 3 rounds)

---

## 10. Code
[QThirtyFour](https://github.com/N-010/core/blob/feature/2025-11-04-QThirtyFour/src/contracts/QThirtyFour.h)
[QReservePool](https://github.com/N-010/core/blob/feature/2025-11-04-QThirtyFour/src/contracts/QReservePool.h)

---

## Glossary

| Term | Definition |
|------|------------|
| **Basis Points (BP)** | 1/100 of 1% (e.g., 500 BP = 5%) |
| **Carry** | Jackpot pool that carries over between rounds |
| **Computor** | Qubic network validator node |
| **Deficit** | Gap between current and target jackpot (Δ) |
| **Epoch** | Qubic time unit (7 days) |
| **FR** | Fast-Recovery mode (jackpot rebuilding system) |
| **Hysteresis** | Delay in state change to prevent rapid toggling |
| **K12** | Kangaroo Twelve cryptographic hash function |
| **k=2, k=3, k=4** | Prize tiers (2, 3, 4 matching numbers) |
| **Overflow** | Unallocated/unawarded prize funds |
| **QPI** | Qubic Programming Interface (contract API) |
| **QRP** | QReservePool contract (reserve management) |
| **QU** | Qubic Unit (base currency) |
| **Rake** | Percentage taken from prize pool |
| **RL** | RandomLottery contract (existing lottery) |
| **Tick** | Qubic block (~1 second) |
| **Winners Block** | Revenue portion allocated to prizes (68%) |

---
