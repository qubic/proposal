# QDuel Contract Proposal

## Overview

QDuel is a duel contract with reusable player balances. A room owner posts a
stake, another player matches it, and the winner receives the pooled amount
minus fees. Owners can keep a deposit on the contract to auto-open a new room
after each duel or timeout.

## Motivation and Goals
- Keep duels lightweight with bounded storage.
- Provide deterministic, verifiable winner selection.
- Allow optional rematching with configurable stake growth.
- Refund or recycle funds when rooms expire.
- Let the team adjust fees and TTL without redeploying.

## Contract Summary
The contract maintains up to 512 active rooms. Each room stores:
roomId, owner, allowedPlayer (optional), amount, closeTimer, and lastUpdate.
A separate users map tracks the room owner profile:
roomId, depositedAmount, locked, stake, raiseStep, and maxStake.

## Key Parameters and Defaults
- minimumDuelAmount = 10,000 QU
- ttlHours = 3
- fee scale = 1000 (1.00% = 10 bps)
- devFeePercentBps = 15 (0.15%)
- burnFeePercentBps = 30 (0.30%)
- shareholdersFeePercentBps = 55 (0.55%)
- asset for shareholders distribution: Random Lottery (RL, assetName 19538)
- tick maintenance period: every 100 ticks

## Room Creation
CreateRoom(owner, allowedPlayer, stake, raiseStep, maxStake):
- invocationReward must be >= minimumDuelAmount and >= stake.
- stake is locked in the room; any extra sent becomes depositedAmount.
- each owner can have only one active room.
- roomId is derived from tick and invocator data; collisions are retried.

## Room Join and Settlement
ConnectToRoom(roomId):
- joining player must send at least room.amount; excess is refunded.
- winner = K12(prevSpectrumDigest XOR player ids XOR tick), LSB picks min/max id.
- if winner derivation fails, both stakes are refunded and the room is closed.
- otherwise, the pooled amount is split into:
  - devFee -> teamAddress
  - burnFee -> burned
  - shareholdersFee -> RL holders (remainder burned)
  - winner -> winner payout

## Fees and Shareholders Distribution

Fee percentages are set in basis points over the scale 1000. The shareholder
portion is rounded down to a value that can be evenly distributed per share.
Distribution uses dividendPerShare = shareholdersFee / NUMBER_OF_COMPUTORS.
Each RL holder receives shares * dividendPerShare, and any remainder is burned.

## Example Duel Settlement (default fees)
- Room amount: 100,000 QU from player1 and 100,000 QU from player2.
- Total pot: 200,000 QU.
- Fees:
  - devFee: 200,000 * 0.15% = 300 QU
  - burnFee: 200,000 * 0.30% = 600 QU
  - shareholdersFee: 200,000 * 0.55% = 1,100 QU (rounded down if needed)
- Winner payout: 200,000 - (300 + 600 + 1,100) = 198,000 QU.

## Auto-Requeue and Deposits
Room owners can keep a deposit on the contract:
- Deposit adds to depositedAmount.
- Withdraw removes free depositedAmount (not locked stakes).
- After a duel, only depositedAmount is considered for auto-requeue.
- After a timeout, depositedAmount + locked stake are considered.

## Next stake is computed as:
- If raiseStep > 1, nextStake = stake * raiseStep.
- nextStake is capped by maxStake if provided.
- If nextStake < minimumDuelAmount, funds are returned and the user profile
  is removed.
- If room creation fails, all available funds are returned and the user profile
  is removed.

## Lifecycle Management
- BEGIN_EPOCH locks the contract until the default RL init time is reached.
- END_TICK runs every 100 ticks to decrement room timers and finalize expired
  rooms.
- Room timers use lastUpdate to avoid large gaps across epochs.

## Admin Controls
- SetPercentFees: teamAddress only; total fee must be < 100%.
- SetTTLHours: teamAddress only; ttlHours must be > 0.
- GetPercentFees, GetTTLHours, GetRooms, GetUserProfile are public queries.

## Limitations and Risks
- Deterministic randomness can be influenced by entities affecting the previous
  spectrum digest; the digest is external to this contract.
- Only room owners are tracked; joiners have no stored profile.
- Auto-requeue depends on available deposits and minimumDuelAmount.
- Fee misconfiguration can shrink winner payouts.

## Conclusion
QDuel provides a compact duel system with adjustable fees, configurable room
TTL, and optional auto-requeue mechanics based on owner deposits. The design
keeps state bounded while supporting repeat play without redeploys.

# Available Options
> Option 0: no, dont allow

> Option 1: yes, allow

# Code
[PR](https://github.com/qubic/core/pull/705)
