# Qubic Proposal: QRaffle — Asset Raffle Extension

## Available Options
 - Option 0: No, do not approve the new Qraffle asset raffles.
 - Option 1: Yes, approve the new Qraffle asset raffles.

## 1. Summary

This proposal extends the existing **QRAFFLE** smart contract with a new feature called **Asset Raffle**: a permissionless mechanism that lets any DAO member raffle a *bundle* of asset shares (issued tokens and/or smart-contract shares) against Qu tickets, with a creator-defined reserve price and a fair, on-chain weighted winner selection at the end of the epoch. If the reserve is met, the bundle is delivered to a single winner and 80% of the Qu pool goes to the creator; if not, every buyer is refunded in full and the bundle is returned to the creator.

The feature is fully additive: no existing QRAFFLE state field, procedure, function, or fee is changed in a backwards-incompatible way. All existing QuRaffle, TokenRaffle, DAO, and QXMR flows continue to work as before. Asset Raffle adds three new user procedures, five new user functions, dedicated per-epoch state, and a new section of `END_EPOCH` settlement.

## 2. Motivation

QRAFFLE today supports two raffle modes:

1. **QuRaffle** — every epoch, members deposit a fixed Qu amount; one winner takes most of the pool.
2. **TokenRaffle** — DAO-approved tokens at fixed entry amounts; one winner takes most of the token pool.

Both modes are **fixed-format**: the entry amount, asset, and structure are decided collectively (DAO-approved per epoch). There is no way for an individual asset holder to spontaneously offer their own asset (or a curated bundle) for raffle with parameters of their choosing.

This is a gap for several use cases that the Qubic ecosystem repeatedly asks for:

- An NFT-like issuer wants to dispose of a specific bundle of issued shares at a fair, transparent price floor.
- A creator wants to combine multiple asset types (e.g., one QX-issued token + a contract-share allocation) into a single prize.
- A user wants a **reserve-protected** sale: "I will release this bundle only if the gross ticket revenue is at least N Qu; otherwise, refund everyone."
- The ecosystem wants a recurring revenue stream for QRAFFLE shareholders and DAO registers from third-party-initiated raffles, without requiring DAO approval for every individual raffle.

**Asset Raffle** fills this gap with a structured, fee-protected, anti-griefing design that re-uses QRAFFLE's existing DAO membership, fee distribution, K12-based RNG, and ring-buffer history pattern.

## 3. Design Goals

1. **Permissionless to create** — any registered DAO member can create a raffle, no per-raffle vote.
2. **Anti-spam** — a non-refundable 500,000 Qu proposal fee per creation, capped at 2 raffles per creator per epoch and 64 active raffles per epoch network-wide.
3. **Reserve-protected** — a creator-set reserve price; if gross ticket revenue net of the 20% fee split would fall below it, all buyers are refunded and the bundle is returned to the creator.
4. **Atomic escrow** — bundle items are escrowed at creation time, with full rollback if any item transfer fails.
5. **Fair winner selection** — weighted by tickets purchased; randomness seeded by previous spectrum + computer digests (un-grindable by computors).
6. **Bounded, deterministic state** — all per-epoch state fully reset at `END_EPOCH`; history kept in a fixed ring buffer.
7. **Aligned economic incentives** — every successful raffle pays QRAFFLE shareholders (dividends), DAO registers, charity, and the fee address; rounding dust is carried forward or paid to creator/registers rather than silently lost.

## 4. User Workflow

```
DAO Member (creator)                     QRAFFLE                          Buyer (any address)
        │                                   │                                   │
        │   createAssetRaffle               │                                   │
        │   (500K Qu fee + bundle escrow)   │                                   │
        ├──────────────────────────────────►│                                   │
        │                                   │  escrow bundle items (atomic)     │
        │                                   │  push half of fee to shareholders │
        │                                   │  push other half to DAO bucket    │
        │                                   │                                   │
        │                                   │       buyAssetRaffleTicket        │
        │                                   │       (numberOfTickets × price)   │
        │                                   │◄──────────────────────────────────┤
        │                                   │                                   │
        │   (optional, if no buyers yet)    │                                   │
        │   cancelAssetRaffle               │                                   │
        ├──────────────────────────────────►│                                   │
        │   (bundle returned; fee burned)   │                                   │
        │                                   │                                   │
        │             ─── END_EPOCH ─────   │                                   │
        │                                   │                                   │
        │                                   │  IF reserve met:                  │
        │                                   │    pick weighted winner           │
        │                                   │    deliver bundle to winner ─────►│ (winner)
        │   creator gets 80% of Qu pool     │                                   │
        │◄──────────────────────────────────┤                                   │
        │                                   │  burn 5% / charity 1%             │
        │                                   │  shareholders 8% (dividends)      │
        │                                   │  registers 5% (per-register)      │
        │                                   │  fee address 1%                   │
        │                                   │                                   │
        │                                   │  IF reserve NOT met:              │
        │                                   │    full Qu refund to buyers ─────►│ (all buyers)
        │   bundle returned to creator      │                                   │
        │◄──────────────────────────────────┤                                   │
```

## 5. Specification

### 5.1 New user procedures

| ID | Name | Caller | Required Qu input | Effect |
|---|---|---|---|---|
| 9 | `createAssetRaffle` | DAO member | ≥ 500,000 Qu | Escrow bundle, register raffle, charge non-refundable proposal fee |
| 10 | `buyAssetRaffleTicket` | Any address | ≥ `numberOfTickets * entryTicketQu` | Add buyer (or top up existing) up to 100 tickets/buyer |
| 11 | `cancelAssetRaffle` | Creator only | 0 | Return bundle, free the per-creator counter; **proposal fee NOT refunded**; only allowed when `numberOfBuyers == 0` |

### 5.2 New user functions (view)

| ID | Name | Purpose |
|---|---|---|
| 10 | `getActiveAssetRaffle` | Read one active raffle's metadata (creator, prices, totals, epoch) |
| 11 | `getActiveAssetRaffleBundleItem` | Read one bundle item (asset + share count) |
| 12 | `getActiveAssetRaffleBuyer` | Read one buyer slot (id + ticket count) |
| 13 | `getEndedAssetRaffle` | Read a settled raffle from the ring buffer by global index |
| 14 | `getAssetRaffleAnalytics` | Read cumulative analytics (totals, success/fail counts, current active count) |

### 5.3 Configuration constants

| Name | Value | Meaning |
|---|---|---|
| `QRAFFLE_ASSET_RAFFLE_PROPOSAL_FEE` | 500,000 Qu | Non-refundable anti-spam fee per raffle creation |
| `QRAFFLE_MAX_ASSET_RAFFLES_PER_EPOCH` | 64 | Concurrent active raffles network-wide |
| `QRAFFLE_MAX_ASSET_RAFFLES_PER_CREATOR` | 2 | Per creator per epoch (decremented on cancel) |
| `QRAFFLE_MAX_ASSETS_PER_BUNDLE` | 4 | Items per bundle |
| `QRAFFLE_MAX_ASSET_TICKET_BUYERS` | 1024 | Distinct buyers per raffle |
| `QRAFFLE_MAX_TICKETS_PER_BUYER` | 100 | Per-buyer cap (anti-griefing) |
| `QRAFFLE_MIN_ASSET_TICKET_AMOUNT` | 1,000,000 Qu | Minimum ticket price |
| `QRAFFLE_MAX_ASSET_TICKET_AMOUNT` | 1,000,000,000,000 Qu | Maximum ticket price (1 trillion) |
| `QRAFFLE_MAX_ENDED_ASSET_RAFFLES` | 8,192 | History ring-buffer size (≈ 128 epochs at full load) |

### 5.4 Economic model — Qu pool split (reserve met)

| Bucket | Share | Mechanism |
|---|---|---|
| Creator | ~80 % | `qpi.transfer(creator, …)`; absorbs per-raffle rounding dust |
| Burn | 5 % | `qpi.burn(…)` |
| Charity | 1 % | `qpi.transfer(charityAddress, …)` |
| Shareholders | 8 % | `qpi.distributeDividends(…)`; rounded to NUMBER_OF_COMPUTORS (676) |
| Registers | 5 % | Per-register equal share; per-raffle dust accumulated, then folded into the DAO bucket and paid out same epoch |
| Fee address | 1 % | `qpi.transfer(feeAddress, …)` |

### 5.5 Reserve check (over-/underflow-safe)

```
reserveMet = (totalTickets > 0)
          && (totalTicketsPaidQu * 80 >= reservePriceQu * 100)
```

Equivalently: `gross >= 1.25 × reservePrice`, because the creator wants `reservePrice` *after* the 20 % fee split. `reservePriceQu` is validated at creation time to satisfy `reservePriceQu ≤ UINT64_MAX / 100` to prevent overflow.

### 5.6 Winner selection

For each successful raffle `i`, a 256-bit seed is computed:

```
baseSeed   = K12( prevSpectrumDigest XOR prevComputerDigest )
raffleSeed = K12( baseSeed XOR (0xA55E7000 + i) )         // per-raffle salt
r          = raffleSeed.u64._0 mod totalTickets
```

The winner is the buyer holding the cumulative ticket range that contains `r` — i.e., weighted by tickets purchased. Tick-level inputs are deliberately excluded from the seed so a computor cannot grind the result.

### 5.7 Reserve-not-met path

If `reserveMet == false` at `END_EPOCH`:

1. For each buyer, refund `tickets × entryTicketQu` in Qu.
2. For each bundle item, return shares to the creator.
3. Record an `EndedAssetRaffleInfo` with `epochWinner = NULL_ID`, `creatorPaidQu = 0`, `reserveMet = 0`.
4. Increment `totalAssetRafflesFailed`.

The 500 K Qu proposal fee is **not** refunded in either path (anti-spam).

### 5.8 Proposal fee distribution (at creation time)

| Bucket | Share | Effect at creation |
|---|---|---|
| QRAFFLE shareholders | 250,000 Qu (50 %) | Added to `epochRevenue`, distributed via `distributeDividends` at next `END_EPOCH` |
| DAO registers | 250,000 Qu (50 %) | Added to `epochAssetRaffleDaoBucket`, distributed per-register at `END_EPOCH` |

Rounding remainders are carried forward in their respective buckets — no silent leakage.

### 5.9 State additions (per-epoch unless noted)

```cpp
// Active raffles
Array<AssetRaffleInfo, 64>                        activeAssetRaffles;
Array<AssetRaffleItem, 64*4>                      activeAssetRaffleItems;       // flat: raffle i at [i*4 .. i*4+bundleSize)
Array<id,     64*1024>                            activeAssetRaffleBuyers;      // flat: raffle i at [i*1024 .. )
Array<uint32, 64*1024>                            activeAssetRaffleBuyerTickets;
uint32                                            numberOfActiveAssetRaffles;

// Per-user O(1) lookups
HashMap<id, BitArray<64>, 65536>                  assetRaffleParticipation;     // bit[i]=1: has bought in raffle i
HashMap<id, Array<uint16, 64>, 65536>             assetRaffleBuyerSlotIndex;    // sentinel 0xFFFF = not present
HashMap<id, uint8, 65536>                         assetRafflesPerCreator;

// History (ring buffer, persistent)
Array<EndedAssetRaffleInfo, 8192>                 endedAssetRaffles;
uint32                                            numberOfEndedAssetRaffles;    // monotonic

// Buckets and analytics (persistent)
uint64                                            epochAssetRaffleDaoBucket;
uint64                                            totalAssetRaffleProposalFees;
uint64                                            totalAssetRaffleCreatorPaid;
uint64                                            totalAssetRaffleRefunded;
uint32                                            totalAssetRafflesCreated;
uint32                                            totalAssetRafflesSucceeded;
uint32                                            totalAssetRafflesFailed;
```

All per-epoch HashMaps and the `numberOfActiveAssetRaffles` / `assetRafflesPerCreator` counters are fully reset at `END_EPOCH`.

### 5.10 Worked example

A creator escrows a bundle of `(QTOKEN × 500,000 shares)`, sets `reservePrice = 100 Qu million`, `entryTicket = 50 Qu million`, and 4 buyers each buy 1 ticket → gross = 200 Qu million.

Reserve check: `200M × 80 = 16,000M`, `100M × 100 = 10,000M`, `16,000 ≥ 10,000` → **met**.

| Bucket | Calculation | Amount (Qu) |
|---|---|---|
| Burn | 200M × 5/100 | 10,000,000 |
| Charity | 200M × 1/100 | 2,000,000 |
| Shareholders (dividends) | 200M × 8/100, rounded to 676 | ≈ 16,000,000 |
| Registers | 200M × 5/100, per-register equal share | 10,000,000 |
| Fee address | 200M × 1/100 | 2,000,000 |
| Creator | 200M − all of the above + dust | ≈ 160,000,000 |

The winner is one of the 4 buyers (uniform among them since each bought 1 ticket); they receive the full bundle (500,000 QTOKEN shares).

## 6. Security Considerations

The implementation has been audited (12 findings, all addressed) and verified by 78 GoogleTest cases (`test/contract_qraffle.cpp`). The most important defensive properties:

1. **Atomic escrow with rollback.** `createAssetRaffle` transfers each bundle item from the creator to `SELF`. If any individual transfer fails, all previously escrowed items are rolled back to the creator before refunding the invocation reward. If a rollback transfer itself fails (e.g., spectrum entry evicted), residual shares stay under contract management for governance recovery — explicitly documented.
2. **Per-creator + global rate limits.** `assetRafflesPerCreator ≤ 2` (decremented on cancel) plus `numberOfActiveAssetRaffles ≤ 64`. Combined with the 500 K Qu non-refundable fee, this caps the per-epoch spam cost.
3. **Bundle validation.** Each item must be a real issued asset (`isAssetIssued`), `numberOfShares > 0`, no internal tokens allowed (QRAFFLE shares with NULL_ID issuer, QXMR with QXMRIssuer), no intra-bundle duplicates.
4. **Reserve overflow guard.** `reservePriceQu ≤ UINT64_MAX / 100` enforced at creation, so `reservePrice × 100` in the END_EPOCH check cannot overflow.
5. **Ticket cost overflow guard.** `entryTicketQu ≤ 1 Trillion` and `tickets ≤ 100` ⇒ cost per call ≤ 1e14, well under `sint64` max. Per-raffle pool ≤ 1e17 (1024 × 100 × 1e12) so `gross × 80` ≤ 8.19e18 (safe).
6. **O(1) duplicate guard.** `assetRaffleBuyerSlotIndex` HashMap maps `(buyer → raffleIndex → buyer-slot position)`, with `0xFFFF` sentinel for "not present". Returning buyers top up their existing slot in O(1) without scanning the buyer list.
7. **Swap-and-pop cancel.** `cancelAssetRaffle` (only allowed when `numberOfBuyers == 0`) compacts the active raffle arrays by moving the last raffle's data — `activeAssetRaffles`, `activeAssetRaffleItems`, `activeAssetRaffleBuyers`, `activeAssetRaffleBuyerTickets` — *and* updates the `assetRaffleParticipation` / `assetRaffleBuyerSlotIndex` of every moved buyer, keeping all parallel arrays in sync.
8. **K12-seeded, un-grindable RNG.** Seed = K12(prevSpectrumDigest XOR prevComputerDigest) XOR per-raffle salt. No tick-level inputs.
9. **Ring-buffer overwrite detection.** `getEndedAssetRaffle` rejects indices that would have been overwritten (`index < numberOfEndedAssetRaffles − 8192`), so callers cannot read stale data.
10. **Rounding-dust safety.** Per-raffle rounding goes to creator. Post-loop register-bucket rounding is folded into the DAO bucket and distributed same epoch; the DAO bucket's own remainder is carried forward, never silently leaked to the contract balance.
11. **`PRE_ACQUIRE_SHARES` policy.** Asset deposits are accepted at zero fee; service fees are collected by user procedures (`depositInTokenRaffle`, `TransferShareManagementRights`). `POST_ACQUIRE_SHARES` credits any unexpected fee to `epochRevenue` defensively.

## 7. Test Coverage

`test/contract_qraffle.cpp` contains **78 GoogleTest cases** for QRAFFLE. The 37 most relevant for Asset Raffle cover:

| Category | Tests |
|---|---|
| Create — validation | `AssetRaffle_CreateBasicSuccess`, `AssetRaffle_CreateValidation`, `AssetRaffle_CreatePerCreatorEpochLimit`, `AssetRaffle_CreateGlobalLimit`, `AssetRaffle_CreateMultiItemBundle`, `AssetRaffle_CreateNonDAOMemberFails`, `AssetRaffle_DuplicateAssetInBundle`, `AssetRaffle_ReservedTokenInBundle`, `AssetRaffle_ZeroOrNegativeSharesInBundle`, `AssetRaffle_ReservePrice_OverflowGuard`, `AssetRaffle_CreateEscrowRollbackOnFailure`, `AssetRaffle_ProposalFeeDistribution` |
| Buy — validation & UX | `AssetRaffle_BuyTicketBasicSuccess`, `AssetRaffle_BuyTicketRefundExcess`, `AssetRaffle_BuyTicketValidations`, `AssetRaffle_BuyTicketCapPerBuyer`, `AssetRaffle_BuyTicketMultipleBuyers`, `AssetRaffle_BuyTicketUnregisteredBuyerAllowed`, `AssetRaffle_BuyTicketSuccessivePurchases`, `AssetRaffle_BuyTicketInvalidAmount` |
| Cancel | `AssetRaffle_CancelNoTickets`, `AssetRaffle_CancelWithTicketsFails`, `AssetRaffle_CancelByNonCreatorFails`, `AssetRaffle_CancelInvalidIndex`, `AssetRaffle_CancelSwapPopMaintainsBuyerState`, `AssetRaffle_CreateAfterCancel_SameEpoch`, `AssetRaffle_ProposalFeeNonRefundable` |
| END_EPOCH settlement | `AssetRaffle_EndEpoch_ReserveMet`, `AssetRaffle_EndEpoch_ReserveNotMet`, `AssetRaffle_EndEpoch_NoBuyers`, `AssetRaffle_EndEpoch_FeeSplit`, `AssetRaffle_EndEpoch_ReserveBoundaryExactlyMet`, `AssetRaffle_EndEpoch_ReserveBoundaryJustMissed`, `AssetRaffle_EndEpoch_MultipleConcurrent`, `AssetRaffle_EndEpoch_StateReset`, `AssetRaffle_EndEpoch_DaoBucketDistributed`, `AssetRaffle_MultipleEpochs_PerCreatorCounterReset` |
| History & view functions | `AssetRaffle_GetActiveAssetRaffle`, `AssetRaffle_GetActiveAssetRaffleBundleItem`, `AssetRaffle_GetActiveAssetRaffleBuyer`, `AssetRaffle_GetEndedAssetRaffle`, `AssetRaffle_GetAnalytics`, `AssetRaffle_EndedRingBufferWrap` |
| End-to-end | `AssetRaffle_FullLifecycle`, `AssetRaffle_AnalyticsCrossCheckMultipleEpochs` |

## 8. Backwards Compatibility

- **Existing procedures (1–8) unchanged.** QuRaffle / TokenRaffle / proposal / vote / register / logout flows are untouched.
- **Existing functions (1–9) unchanged.** Existing API responses are identical (with the exception of `getRegisters`, which was relaxed to return partial trailing pages with NULL_ID sentinels instead of `INVALID_OFFSET_OR_LIMIT` — see "Other improvements bundled" below).
- **New procedure IDs (9–11) and function IDs (10–14)** are appended; no existing ID is reused.
- **State additions** are appended to `StateData`. Persistent fields (history ring buffer, cumulative analytics) start zeroed on construction.
- **No fee changes for any existing path.** All existing tests continue to pass.

## 9. Other improvements bundled with this proposal

While extending the contract for Asset Raffle, the following hardening / cleanup changes were applied to QRAFFLE itself. They are non-breaking but worth noting in the proposal:

- **F-01 — QXMR distribution unconditional.** QXMR shareholder distribution was hoisted out of the QuRaffle conditional, so logout-fee revenue accumulated in `epochQXMRRevenue` is paid out every epoch (regardless of QuRaffle activity).
- **F-02 — Register-bucket dust carry-forward.** Per-raffle register-bucket remainders are folded into the DAO bucket and either paid out same epoch or carried forward; previously they were silently retained in the contract balance.
- **F-03 — Dead `shareholdersList` removed.** A populated-but-never-read `HashSet<id, 1024>` saved ~2 MB of state and one full possessor iteration per epoch.
- **F-05 — `getRegisters` returns partial pages.** Callers no longer need to know `numberOfRegisters` in advance.
- **F-10 — `AssetRaffleEndedLogger._raffleIndex`** now logs the monotonic global ended-raffle index (matches what `getEndedAssetRaffle` expects) instead of the per-epoch slot.

## 10. Deployment

Because this is an upgrade of an already-deployed contract (QRAFFLE, index 19), the standard contract-upgrade flow applies:

1. PR `feature/qraffle-asset` merged into `qubic/core` `develop`, with all 78 tests passing and the Qubic Contract Verification Tool clean.
2. A computor operator posts a GQMPROP "yes/no" inclusion proposal for the new QRAFFLE binary, with a permalink to this document.
3. If quorum (≥ 451 yes votes and yes > no) is reached in epoch *N*, the new contract code ships in the next release; epoch *N+1* runs the upgraded code in the same construction context (no second IPO — QRAFFLE was IPO'd in epoch 191 originally).
4. First production use of Asset Raffle is therefore from epoch *N+1*.

No state migration is required: the new state fields are appended and start zeroed.

## 11. Source code

Final source: [`src/contracts/QRaffle.h`](../src/contracts/QRaffle.h) at the head of branch `feature/qraffle-asset` (referenced commit SHA to be pinned in the GQMPROP description at submission time).

Tests: [`test/contract_qraffle.cpp`](../test/contract_qraffle.cpp).

## 12. Acceptance Criteria

The computor community is asked to vote **yes** on this proposal if, after independent review, the following statements all hold:

- The contract upgrade compiles and passes the Qubic Contract Verification Tool.
- All 78 GoogleTest cases in `test/contract_qraffle.cpp` pass on a clean build of `feature/qraffle-asset`.
- The state-size addition is acceptable (the new state fits well within the per-contract budget; concrete bytes will be reported by `Qubic.exe` at construction).
- The economic model (Section 5.4 / 5.8) is acceptable to the network: 80 % to creator, 20 % to the existing QRAFFLE fee buckets, plus a non-refundable 500 K Qu proposal fee split 50/50 between shareholders and DAO registers.
- The security review notes in Section 6 have no outstanding objections.

If any acceptance criterion fails, the proposal should be voted **no**; the changes can be re-submitted after addressing the concerns.
