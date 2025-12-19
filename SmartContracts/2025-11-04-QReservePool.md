# QReservePool (QRP)

## Available Options
> Option 0: no, dont allow

> Option 1: yes, allow

## Table of Contents

1. [Introduction](#1-introduction)
2. [Goals and Non-Goals](#2-goals-and-non-goals)
3. [Core Model](#3-core-model)
4. [Interface Summary](#4-interface-summary)
5. [Access Control](#5-access-control)
6. [Integration: QThirtyFour (QTF)](#6-integration-qthirtyfour-qtf)
7. [Future Integrations: RL-family Projects](#7-future-integrations-rl-family-projects)
7. [Code](#8-code)

---

## 1. Introduction

QReservePool (QRP) is a shared on-chain reserve vault implemented in `src/contracts/QReservePool.h`.
Its purpose is to provide a simple, reusable backstop reserve that allowlisted smart contracts can draw from when their own per-epoch cashflow is insufficient to meet obligations.

At the current stage, QRP is required by `src/contracts/QThirtyFour.h` (QTF) as a reserve source for payout floors and jackpot carry rebuild scenarios.
In the future, other RandomLottery-style projects are expected to use this same pool instead of deploying separate reserve contracts.

---

## 2. Goals and Non-Goals

### Goals

- **Centralize reserve management** in a single contract.
- **Restrict withdrawals** to allowlisted smart contracts only.
- **Keep the surface minimal** and easy to audit.
- **Enable reuse** by future RL-family contracts via allowlisting.

### Non-Goals

- Per-consumer accounting or “sub-balances” per project.
- Rate limiting, per-epoch quotas, or per-winner limits.
- Enforcement of any consumer economics/spec rules (those belong to consumer contracts).

---

## 3. Core Model

### Reserve Definition

QRP treats its available reserve as the net balance of its entity:

- `availableReserve = incomingAmount - outgoingAmount` (clamped to >= 0)

### Withdrawal Model

- A consumer contract requests a withdrawal by calling `GetReserve`.
- If the caller is allowlisted and enough reserve is available, QRP transfers the requested amount to the invocator.

---

## 4. Interface Summary

| Method | Type | Purpose |
|--------|------|---------|
| `GetReserve` | Procedure | Withdraw reserve to an allowlisted contract (caller = invocator). |
| `AddAvailableSC` | Procedure | Owner-only: add a contract index to the allowlist. |
| `RemoveAvailableSC` | Procedure | Owner-only: remove a contract index from the allowlist. |
| `GetAvailableReserve` | Function | Read available reserve (net balance). |
| `GetAvailableSC` | Function | Read allowlisted smart contracts. |

---

## 5. Access Control

### Allowlisted Withdrawals

- `GetReserve` must be callable only by allowlisted contract IDs.
- End users (wallets) cannot withdraw directly.

### Owner Governance

- The allowlist is maintained via owner-only procedures:
  - `AddAvailableSC`
  - `RemoveAvailableSC`

---

## 6. Integration: QThirtyFour (QTF)

QRP is currently needed for QTF (`src/contracts/QThirtyFour.h`) to support:

- **Payout floors backstop:** when epoch revenue is too low to meet minimum guaranteed payouts.
- **Carry/jackpot rebuild support:** when QTF needs to reseed/rebuild carry after a k=4 jackpot win (subject to QTF’s own safety rules).

QRP itself does not encode QTF-specific limits. QTF must enforce its own caps and policies before calling `GetReserve`.

---

## 7. Future Integrations: RL-family Projects

The long-term intent is for multiple RL-derived contracts (RandomLottery-style) to share the same reserve pool:

- New consumer onboarding should be done by adding the contract to QRP allowlist (by contract index).
- Each consumer contract must implement its own safety caps, governance constraints, and usage policy.

This approach avoids deploying (and auditing) a separate reserve contract for every RL-family project.

---

## 8. Code

[QThirtyFour](https://github.com/N-010/core/blob/feature/2025-11-04-QThirtyFour/src/contracts/QThirtyFour.h)

[QReservePool](https://github.com/N-010/core/blob/feature/2025-11-04-QThirtyFour/src/contracts/QReservePool.h)

---
