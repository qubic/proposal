# Smart Contract Proposal - L2 HashStore - 2025-07-08

To enable decentralized anchoring of off-chain computation states, this proposal suggests the deployment of a lightweight Smart Contract called **HashStore**. This contract stores 32-byte hashes on the QUBIC chain, representing external computation states.

## Proposal

Allow the deployment of the Smart Contract **HashStore** on QUBIC.  
The contract enables external systems (e.g., Layer 2 EVM execution environments) to submit cryptographic proofs of state, ensuring auditability and traceability on-chain.

This is an **infrastructure smart contract**.  
It will **not generate any revenue** for shareholders.  
All invocation rewards are **burned**.

### Voting Options

> Option 1: Yes, allow  
> Option 2: No

---

## Purpose

This contract is part of a broader initiative to build an **EVM-compatible Layer 2** (L2) on top of QUBIC. While the contract logic runs off-chain, this Smart Contract allows developers to publicly anchor the final state of those executions.

Use cases include:
- Anchoring state roots from off-chain EVM contract execution.
- Verifying state changes through audit trails.
- Using QUBIC as a **settlement layer**.

---

## Contract Behavior

- `SetHash`: Saves a 32-byte hash to contract state (overwriting the previous one).
- `GetHash`: Returns the currently stored hash.
- The contract keeps **only the most recent hash**, ensuring minimal on-chain storage.

---

## Technical Implementation

```cpp
struct HASHSTORE {
    Array<uint8, 32> hash;
};

struct SetHash_input { Array<uint8, 32> hash; };
struct SetHash_output {};

struct GetHash_input {};
struct GetHash_output { Array<uint8, 32> hash; };

PUBLIC_PROCEDURE(SetHash)
{
    for (uint32 i = 0; i < 32; i += 1) {
        state.hash[i] = input.hash[i];
    }
}

PUBLIC_FUNCTION(GetHash)
{
    for (uint32 i = 0; i < 32; i += 1) {
        output.hash[i] = state.hash[i];
    }
}

void REGISTER_USER_FUNCTIONS_AND_PROCEDURES() {
    REGISTER_USER_PROCEDURE(SetHash, 1);
    REGISTER_USER_FUNCTION(GetHash, 2);
}
```

---

## Security and Design

- Open to all users for writing hashes.
- No funds are stored or transferred — no financial risk.
- Each invocation requires a reward (e.g., 1 mio QUBIC) that is burned, preventing spam.
- Stateless beyond the last saved hash, enabling lightweight and efficient use.

---

## Rationale

This contract establishes a minimal yet critical foundation for off-chain proof anchoring and traceable settlement. It enables:

- External EVM-based systems to record state transitions.
- Developers to build rollups, zk-based systems, or lightweight bridges.
- A future-proof design that integrates smoothly with decentralized compute infrastructure.

---

## Possible Extensions (Future Proposals)

- Store multiple hashes (ring buffer or historical log).
- Merkle Root storage for multiple contracts or accounts.
- Submit zero-knowledge or recursive proofs tied to state changes.

---

## Example Use Case

1. A user deploys the FLIPPER contract off-chain via a custom EVM backend (e.g., Express + JS engine).
2. The backend executes flip(), changing contract state.
3. It hashes the new EVM state and submits that hash via SetHash to the QUBIC HashStore.
4. The hash becomes publicly visible and anchored in the QUBIC ledger. 