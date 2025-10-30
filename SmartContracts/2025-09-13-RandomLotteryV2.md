# RL-02: Flexible Ticketing & Epoch Configuration

**Author:** RandomLottery (RL) Team
**Goal:** Enable more lottery formats without changing code.

## Available Options

> Option 0: no, don’t allow

> Option 1: yes, allow

## Summary

We propose four product changes to RL:

1. **Unlimited tickets per account** — a single account can buy any number of tickets within one epoch.
2. **Configurable price for the next epoch** — set the price in advance; it takes effect at the start of the following epoch.
3. **Configurable number of draws per epoch** — run one or multiple draws within the same epoch.
4. **Configurable number of winners** — each draw can have one or multiple winners.

Together, these changes enable “premium,” “standard,” and “dynamic” formats while preserving RL’s transparency.

## Motivation

* **Flexibility:** tune price, pacing, and winner count to match demand, promotions, and events.
* **Higher engagement:** wins happen more often, with varied play styles.
* **Operational simplicity:** all adjustments are planned per epoch, with no code redeploys.

## What Changes for Players

* You can buy **multiple tickets** in a single epoch.
* The site will show both the **current price** and the **scheduled next-epoch price**.
* An epoch may include **multiple draws**, and **each draw can have multiple winners** (the winner share is divided among them).

## Benefits

* **Larger prize pools:** power users can grow the bank by buying more tickets.
* **Different play styles:** from rare “premium” draws to frequent “dynamic” ones with several winners.
* **Transparency:** all changes are announced in advance and take effect from the next epoch.

## Example Formats

* **Premium:** higher ticket price, 1 draw, 1 winner — maximized single payout (e.g., every 3–4 epochs).
* **Dynamic:** lower ticket price, 2–3 draws per epoch, **2–5 winners** in each — frequent wins and more happy players.
* **Community Weeks:** temporary price reductions + more winners to attract new players.

## Code change
For detailled code changes see the pull request below:
https://github.com/qubic/core/pull/586

