# Proposal for Quottery v2 update

## Proposal
Allow Deployment of the new Quottery Smart Contract in EP201 on Qubic

### Available Options

> Option 0: No — do not allow deployment

> Option 1: Yes — allow deployment

## Overview
Quottery (QTRY) is a high-performance, decentralized prediction market and binary options exchange designed specifically for the Qubic ecosystem. Unlike legacy prediction platforms, QTRY operates as a fully on-chain order book exchange, allowing users to trade positions on future outcomes with zero counterparty risk.

Built using highly optimized C++ for the QPI, QTRY leverages Qubic’s unique "tick" architecture to provide near-instant matching, automated settlement, and a trustless dispute resolution mechanism powered by the network's Computors. It represents a significant step forward in utility for the Qubic network, offering a distinct financial instrument that drives transaction volume and burns QUs.

## Motivation and Goals

### Motivation
Current decentralized prediction markets often suffer from high latency, excessive gas fees, or centralized oracle points of failure. The Qubic ecosystem currently lacks a native financial derivative layer that can handle high-frequency trading of binary outcomes (Yes/No events) without bloating the state. QTRY addresses this by implementing a lightweight, "set-and-forget" architecture that cleans up its own state, ensuring long-term sustainability.

### Goals
* **Trustless Settlement:** To remove the need for a centralized clearinghouse by using atomic smart contract swaps for all trades.
* **Economic Utility:** To create a consistent "sink" for QUs through trading fees, event creation fees, and an automated burn mechanism.
* **Fair Dispute Resolution:** To utilize the consensus of Qubic Computors as the ultimate "Supreme Court" for resolving controversial event outcomes, ensuring fairness beyond the control of the platform operator.
* **High Efficiency:** To demonstrate the capabilities of QPI by managing up to 2 million user positions and thousands of concurrent events without impacting network tick times.

## Contract Summary
The QTRY smart contract is a self-contained financial engine. Its core architecture revolves around a dual-state order book that manages liquidity for binary options (Option 0 vs. Option 1).

* **Matching Engine:** The contract features an embedded matching engine that supports three distinct trade types:
    * **Direct Match:** Standard Ask vs. Bid matching.
    * **Merge (Liquidity Exit):** Users holding opposite positions can "merge" them to exit the market and recover collateral.
    * **Mint (Liquidity Entry):** Two users with opposing views can simultaneously fund a new position, creating liquidity for the market.
* **Automated Lifecycle:** Events move strictly through a state machine: `Active` → `Trading Closed` → `Result Published` → `Dispute Window` → `Finalized` → `Archived`.
* **State Management:** The contract includes aggressive, automated garbage collection in the `END_EPOCH` routine. It ensures that finalized events and settled positions are permanently removed from memory, preventing state bloat—a critical feature for maintaining low operational costs on Qubic.
* **Security:** The contract enforces strict checks on amounts, prices, and dates, and includes "anti-spam" deposits for creating events or raising disputes.

## Governance
QTRY introduces a multi-layered governance model that balances operational efficiency with decentralized oversight.

* **The Game Operator (GO):** A designated entity (initially the developer team) responsible for the day-to-day creation of events and publishing of initial results. The GO must stake capital to perform actions, incentivizing accurate reporting.
* **The Computors (Validators):** Qubic Computors serve as the ultimate oracle. If users disagree with the Game Operator’s published result, they can initiate a **Dispute**. Computors then vote on the correct outcome. If the GO is found to be malicious or incorrect, they are penalized, and the Computors update the result directly on-chain.
* **Shareholder Proposals:** Holders of the QTRY asset have direct control over the protocol's economic parameters. Through a built-in proposal system, shareholders can vote to:
    * Adjust fee rates (Operation Fee, Shareholder Fee, Burn Fee).
    * Update the "Anti-Spam" and "Deposit" amounts.
    * **Replace the Game Operator** entirely if they become inactive or malicious.

## Dividend Distribution
QTRY implements a robust, automated revenue-sharing model designed to benefit all stakeholders in the Qubic ecosystem.

* **Revenue Streams:** The contract collects revenue from:
    * **Trading Fees:** A percentage fee (default 1.5%) charged on every matched trade.
    * **Hosting Fees:** A daily fee charged to the Game Operator for keeping an event active.
* **Distribution Logic:** At the end of every epoch (approx. every week), the contract automatically calculates the total accumulated revenue and distributes it as follows:
    1.  **Deflationary Burn:** A configurable percentage (e.g., 10%) is permanently burned (removed from circulation), reducing the total supply of QUs and benefiting all Qubic holders.
    2.  **Shareholder Dividends:** The majority of the net revenue is distributed pro-rata to all holders of QTRY shares. This creates a yield-generating asset within the Qubic ecosystem.
    3.  **Operational Costs:** A portion is allocated to the Game Operator to cover server costs, data feed subscriptions, and development.

This "Push" distribution model (automated at `END_EPOCH`) ensures that dividends are paid out regularly and transparently, without requiring users to manually claim or pay fees to receive their rewards.

## Technical implementation
https://github.com/qubic/core/pull/761
