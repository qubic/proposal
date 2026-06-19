# Proposal: Upgrade Qbay Smart Contract — Burn Collection Creation CFB Fees

## Proposal

Upgrade the existing QBAY smart contract so that CFB tokens paid when creating a collection are burned instead of collected by the marketplace.

## Available Options

Option 0: No, do not approve the Qbay revisions.

Option 1: Yes, approve the Qbay revisions (burn collection creation CFB fees).

## Context

QBAY (contract index 12) is the Qubic NFT marketplace smart contract. When users create a collection, they pay a CFB fee based on the collection size. Today, that fee is transferred to the QBAY contract and tracked in the `earnedCFB` state variable. At the end of each epoch, `earnedCFB` is transferred to the marketplace owner along with other CFB marketplace revenue (NFT sales, auctions, offers, etc.).

This proposal changes only the handling of **collection creation** CFB fees. Users will still pay the same CFB amounts to create collections, but those tokens will be sent to the burn address (`NULL_ID`) and permanently removed from circulation. All other CFB fee flows remain unchanged.

## Summary of Changes

### Collection Creation Fee Handling

**Current behavior** (`createCollection` procedure in `core/src/contracts/Qbay.h`):

- When a non-marketplace-owner user creates a collection, the required CFB fee is transferred to the QBAY contract (`SELF`).
- The fee amount is added to `state.earnedCFB`.
- At `END_EPOCH`, collection creation fees are included in the CFB payout to the marketplace owner.

**Proposed behavior:**

- Collection creation CFB fees are transferred to `NULL_ID` (the burn address) instead of `SELF`.
- Collection creation fees are **no longer** added to `earnedCFB`.
- CFB tokens paid for collection creation are permanently burned.

### What Does Not Change

- Collection creation fee amounts remain the same (100–2000 USD-equivalent fee units, depending on collection volume; actual CFB paid is `fee × priceOfCFB`).
- Users must still possess sufficient CFB to create a collection.
- The marketplace owner can still create collections without paying CFB.
- CFB fees from NFT marketplace activity (sales, asks, auctions, drop mints, etc.) are still collected in `earnedCFB` and distributed to the marketplace owner at `END_EPOCH`.
- QU-based fees and shareholder dividend distribution are unaffected.

## Collection Creation Fee Schedule (unchanged)

The fee values below are **USD-equivalent** fee units, not CFB token amounts. The contract uses the oracle price `priceOfCFB` (CFB per 1 USD) to calculate the actual CFB shares required:

```
CFB shares transferred = fee × priceOfCFB
```

| Collection capacity | Fee (USD equivalent) |
|---------------------|----------------------|
| 2–200 NFTs          | $100                 |
| 201–1,000 NFTs      | $200                 |
| 1,001–2,000 NFTs    | $400                 |
| 2,001–3,000 NFTs    | $600                 |
| 3,001–4,000 NFTs    | $800                 |
| 4,001–5,000 NFTs    | $1,000               |
| 5,001–6,000 NFTs    | $1,200               |
| 6,001–7,000 NFTs    | $1,400               |
| 7,001–8,000 NFTs    | $1,600               |
| 8,001–9,000 NFTs    | $1,800               |
| 9,001–10,000 NFTs   | $2,000               |

Because `priceOfCFB` is set by the marketplace owner via `settingCFBAndQubicPrice` and reflects the current CFB/USD rate, the actual number of CFB tokens burned per collection creation varies with the oracle price.

## Rationale

Burning collection creation CFB fees reduces CFB supply over time as new collections are created on Qbay. This aligns collection creation with a deflationary token model rather than routing those fees to marketplace revenue. Marketplace revenue from ongoing NFT trading activity (sales, auctions, offers) continues to flow to the marketplace owner as before.

## Technical Implementation

Core implementation: https://github.com/qubic/core/pull/922

### Code change in `createCollection`

In `core/src/contracts/Qbay.h`, the `createCollection` procedure is updated as follows:

```c++
// Before:
qpi.transferShareOwnershipAndPossession(QBAY_CFB_NAME, state.get().cfbIssuer, qpi.invocator(), qpi.invocator(), locals.fee * state.get().priceOfCFB, SELF);
state.mut().earnedCFB += locals.fee * state.get().priceOfCFB;

// After:
qpi.transferShareOwnershipAndPossession(QBAY_CFB_NAME, state.get().cfbIssuer, qpi.invocator(), qpi.invocator(), locals.fee * state.get().priceOfCFB, NULL_ID);
```

Passing `NULL_ID` as the destination burns the CFB shares per the QPI specification (`qpi.h`: *"Pass NULL_ID to burn shares"*).

### Upgrade notes

- In-place upgrade of the existing QBAY smart contract.
- No state migration required; only the `createCollection` procedure logic changes.
- Existing `earnedCFB` balance at the time of upgrade is unaffected and will be distributed normally at the next `END_EPOCH`.

## Recommendation

Vote Option 1 to approve burning CFB collection creation fees on Qbay.
