# QIP Upgrade Proposal 

**Proposer:** NINEISTEN

**Date:** June 27, 2026

## Voting Options
- **Option 1**: **Approve the QIP upgrades**
- **Option 0**: Reject the upgrades 

## Summary
This proposal requests Quorum approval to integrate the QIP (Qubic ICO Portal) upgrades from the following update:


## Problem / Opportunity
The current QIP contract has limitations in ICO index management, vesting lifecycle handling, and exchange-order conflict detection. Active ICO tracking relies on a compact sequential counter, which complicates slot reuse and cleanup. Vesting fund returns and token distribution contain off-by-one epoch errors. There is no protection against creating an ICO while open QX ask orders exist for the same asset. The `getBuyerInfo` function lacks vesting status details needed by frontends and integrators.


**Key Upgrades Included:**

- **Increased ICO Capacity**:
  - `QIP_MAX_NUMBER_OF_ICO` raised from **1024** to **16384** concurrent active ICO slots.

- **New Error Codes**:
  - `QIP_invalidIssuer` (15): invocator is not the token issuer.
  - `QIP_qxAskOrderFound` (16): open QX ask orders exist for the asset being listed.

- **State Management Refactor (`StateData`)**:
  - Replaced `numberOfICO` with:
    - `HashSet<uint64, QIP_MAX_NUMBER_OF_ICO> activeIcoIndexes` — tracks currently active ICO indices.
    - `uint32 currentIcoIndex` — monotonically increasing slot counter for new ICOs.
  - ICO creation writes to `currentIcoIndex` and registers the index in `activeIcoIndexes`.
  - Completed ICOs are removed from `activeIcoIndexes` (data remains in the `icos` array).
  - `activeIcoIndexes.cleanupIfNeeded()` added at end of epoch processing.

- **Enhanced `createICO` Validation**:
  - **Issuer check**: `qpi.invocator()` must equal `input.issuer`; otherwise returns `QIP_invalidIssuer`.
  - **QX integration**: calls `QX::AssetAskOrders` via `CALL_OTHER_CONTRACT_FUNCTION`; rejects ICO creation if any open ask order exists (`numberOfShares != 0`), returning `QIP_qxAskOrderFound`.
  - Overflow check now uses `activeIcoIndexes.population()` instead of `numberOfICO`.
  - Share transfer validation runs only when `output.returnCode == 0` (after all prior checks pass).

- **Enhanced `getBuyerInfo` Function**:
  - Output fields renamed: `toReceive` → `tokensToReceive`, `received` → `receivedTokens`.
  - New output fields: `isVested`, `remainingVestingPeriod`.
  - `remainingVestingPeriod` is computed when ICO is vested, buyer has not returned funds, and vesting is still in progress.
  - Removed upper-bound check on `indexOfICO` (relies on direct ICO array lookup).

- **Updated ICO Lookup in Procedures**:
  - `buyToken`, `returnFunds`: validate ICO existence via `activeIcoIndexes.contains(input.indexOfICO)` instead of `indexOfICO >= numberOfICO`.
  - `TransferShareManagementRights`: iterates `activeIcoIndexes` HashSet instead of sequential `numberOfICO` loop; blocks share-management transfer only for **non-vested** active ICOs matching the asset (`&& !locals.ico.isVested`).

- **`returnFunds` Bug Fix**:
  - Vesting penalty epoch offset corrected from `startEpoch - 3` to `startEpoch - 2` in the return amount calculation.

- **`END_EPOCH` Refactor**:
  - Iterates `activeIcoIndexes` instead of sequential index loop.
  - Non-vested ICO cleanup: uses `activeIcoIndexes.removeByIndex()` instead of swap-with-last array compaction.
  - Sets `remainingAmountForPhase3 = 0` after phase 3 ends.
  - **Vesting cancellation logic**: if all buyers have returned funds (`isVestingCancelled` remains true), skips QU distribution to address2–address10 and returns unreceived tokens to the ICO creator via `transferShareOwnershipAndPossession`.
  - Handles returned buyers separately: accumulates `tokenAmountToReturn` for unreceived token amounts.
  - Last vesting distribution epoch adjusted: final token payout triggers at `startEpoch + vestingPeriod + 1` (was `+ 2`).
  - On final epoch: sets `buyerInfo.received = buyerInfo.toReceive` and keeps entry in HashMap (instead of removing buyer record).
  - QU address transfers (address2–address10) execute only when vesting is **not** cancelled.
  - ICO removed from `activeIcoIndexes` at `startEpoch + vestingPeriod + 2`.

## Benefits
- Supports up to 16× more concurrent active ICOs without index collision issues.
- Prevents ICO creation conflicts with existing QX exchange orders for the same asset.
- Ensures only the token issuer can launch an ICO for their asset.
- Correct vesting epoch math for fund returns and token distribution.
- Graceful handling of fully cancelled vesting (all investors returned funds).
- Richer `getBuyerInfo` output for UI/integrator vesting progress display.
- Cleaner active-ICO lifecycle via HashSet-based tracking and cleanup.

## Implementation
https://github.com/qubic/core/pull/932/changes/03b5708c524ca80b87524ddc400cc1234d8cfb09
