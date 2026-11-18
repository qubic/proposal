# Smart Contract Update - QRaffle Initial Register Wallets

## Proposal
Allow the QRaffle Smart Contract (contract index 19) to replace three of its five initial DAO register wallets.

### Available Options

> Option 0: No — keep the current register wallets

> Option 1: Yes — replace initial registers 1, 2 and 3 at the start of epoch 241

---

## Executive Summary

QRaffle has five initial DAO register wallets. They were written into the contract when it was constructed in epoch 192 and cannot be changed by any transaction. Three of them are no longer safe to use. One was hacked, and two were generated from weak seeds.

This update replaces those three wallets with new ones. The replacement runs once, at the start of epoch 241.

- **Only three wallet IDs change.** Registers 4 and 5, the charity address, the fee address, every fee and every procedure stay exactly as they are
- **No state layout change.** No fields are added or removed, so the contract state keeps its size and needs no migration
- **No funds move.** The swap only changes who holds the three seats. Initial registers locked no deposit, so there is nothing to refund or transfer
- **The DAO keeps five initial registers.** The number of DAO members does not change

---

## Why it is needed

QRaffle's initial registers are permanent DAO members. Each one:

- receives an equal share of the 5% register fee from every QuRaffle, Token Raffle and Asset Raffle
- receives an equal share of the DAO half of every Asset Raffle creation fee
- can submit and vote on Token Raffle proposals
- can create Asset Raffles
- can never log out (`logoutInSystem` rejects all five initial registers)

Because the seats are permanent and hard-coded, the contract has no way to retire a wallet whose key is compromised. Today:

- **One wallet was hacked.** Whoever holds its key can vote and propose in the QRaffle DAO, and collects that seat's share of all register revenue every epoch
- **Two wallets have weak seeds.** One of them, register 1 (`ILNJXVHAUXDGGBTTUOITOQGPAYUCFTNCPXDKOCPUOCDOTPUWXBIGRVQDLIKC`), is already on core's frozen weak-seed list (`src/frozen_addresses.h`, frozen from tick 87,500,000). The network drops every transaction it sends. QRaffle still pays register revenue into it every epoch, and that Qu can no longer be spent

The only fix is a contract update.

---

## The change

| Slot | Current wallet (removed) | New wallet |
|---|---|---|
| Initial register 1 | `ILNJXVHAUXDGGBTTUOITOQGPAYUCFTNCPXDKOCPUOCDOTPUWXBIGRVQDLIKC` | `BOYWOYXADZOTEESXCMVSOZQFZOUBJSDIIRKHITTFIDLTKAGIOMKIXQFEHHMG` |
| Initial register 2 | `LSDAACLXXGIPGGLSOCLMVAYLNTGDVBNOSSYEQDRKXDYWBCGJIKCMZKMFUEBH` | `FIHOTEXIOKEMWAWQJTOPZEBAJTGAAYOHBQFWMXVBADQHHNGGUJHLSDYGMJVJ` |
| Initial register 3 | `GHGRLWSXZXWDAAOMTXQYUPRLPNKCWGHAEFIRJIZAKCAUDGNMCDEQROQBVOPB` | `UZSTADSMJDWKYCAYIGXLGNUXPNOCIXMGQIULGZQRJGIMNDBGKAELVTTEDYXJ` |
| Initial register 4 | `EUONAZJUAGVDCEIBAHJETGUUHMNDJCSETTQVGYFHMDPXTALDYUVEPFCAWVNF` | unchanged |
| Initial register 5 | `SLCJCCUXGKNVADFBEAYVLSOBZPABHKSGMHWHSHGGBAPJWFVOKZJPFLXDRSVA` | unchanged |

### How it runs

The contract gains a `BEGIN_EPOCH` procedure that does its work only when the epoch is 241. For each of the three slots it:

1. removes the old wallet from the DAO register list
2. adds the new wallet as an initial register (no deposit, cannot log out)
3. stores the new wallet in the slot (`initialRegister1`, `2` or `3`), so the logout protection applies to the new wallet and no longer to the old one

`BEGIN_EPOCH` runs at the first tick of epoch 241. That is directly after `END_EPOCH` of epoch 240 has paid out all raffles and cleared all per-epoch state, so the old wallets have no open votes, proposals or Asset Raffles that carry over.

From epoch 241 the old wallets are ordinary addresses. They receive no register revenue, cannot vote or propose, and cannot create Asset Raffles. In every later epoch the `epoch == 241` check is false and `BEGIN_EPOCH` does nothing. The inactive code can be removed in a later release.

The same three IDs are also changed in `INITIALIZE`. That procedure only runs when the contract is constructed, so this changes nothing on mainnet. It keeps fresh test deployments consistent with the live contract.

### What does not change

- Registers 4 and 5, the charity address, the fee address and the QXMR issuer
- All fees and revenue splits
- All procedure and function IDs, their inputs and their outputs
- The state layout and state size. No state-change entry is needed in `contract_def.h`
- Paid DAO members and their deposits

---

## Safety

- **Number of members stays the same.** Each slot removes one register and adds one, so `numberOfRegisters` is unchanged. If a removed wallet is already missing, or a new wallet is already a member, the counter is only adjusted for what actually changed
- **The new wallets must not already be paid DAO members.** If a new wallet were already registered with a 1B Qu or QXMR deposit, it would keep that membership type but become an initial register, which cannot log out, and its deposit would be locked. The QRaffle team confirms that none of the three new wallets is a registered DAO member
- **The update runs exactly once.** The swap is tied to a single epoch, and `BEGIN_EPOCH` runs once per epoch
- **The update cannot block settlement.** The swap makes no transfers, calls no other contract, and runs in `BEGIN_EPOCH`, which runs regardless of the contract's execution fee reserve

---

## Timeline

1. **Epoch 240:** computors vote on this proposal
2. **If accepted:** the change ships in the epoch-241 core release
3. **First tick of epoch 241:** the swap runs automatically

If the release that contains this change goes live later than epoch 241, the trigger epoch in the code will be moved to that release's first epoch. Nothing else changes.

---

## How to check it after epoch 241

- `getRegisters` (function 1) with `offset = 0`, `limit = 20` lists the three new wallets and none of the three old ones
- `getAnalytics` (function 2) shows the same `numberOfRegisters` as at the end of epoch 240
- Calling `logoutInSystem` from one of the new wallets returns `QRAFFLE_INITIAL_REGISTER_CANNOT_LOGOUT` (12)

---

## Technical implementation

**Developer:** [double-k-3033](https://github.com/double-k-3033)

From: branch [`fix/qraffle-wallet`](https://github.com/qubic/core/tree/fix/qraffle-wallet) of `qubic/core`, file `src/contracts/QRaffle.h`.

The code below is added after `INITIALIZE()`:

```c++
	// Replace one bootstrap register in the live registers map: drop the old ID, add the new ID as an initial register (tokenType 0).
	struct replaceInitialRegister_input
	{
		id oldRegister;
		id newRegister;
	};
	struct replaceInitialRegister_output
	{
	};

	PRIVATE_PROCEDURE(replaceInitialRegister)
	{
		if (state.get().registers.contains(input.oldRegister))
		{
			state.mut().registers.removeByKey(input.oldRegister);
			state.mut().numberOfRegisters--;
		}
		if (!state.get().registers.contains(input.newRegister))
		{
			state.mut().registers.set(input.newRegister, 0);
			state.mut().numberOfRegisters++;
		}
	}

	struct BEGIN_EPOCH_locals
	{
		replaceInitialRegister_input replaceInput;
		replaceInitialRegister_output replaceOutput;
	};

	BEGIN_EPOCH_WITH_LOCALS()
	{
		// One-time swap of initial register wallets 1-3 on the already-deployed contract.
		if (qpi.epoch() == 241)
		{
			locals.replaceInput.oldRegister = state.get().initialRegister1;
			locals.replaceInput.newRegister = ID(_B, _O, _Y, _W, _O, _Y, _X, _A, _D, _Z, _O, _T, _E, _E, _S, _X, _C, _M, _V, _S, _O, _Z, _Q, _F, _Z, _O, _U, _B, _J, _S, _D, _I, _I, _R, _K, _H, _I, _T, _T, _F, _I, _D, _L, _T, _K, _A, _G, _I, _O, _M, _K, _I, _X, _Q, _F, _E);
			CALL(replaceInitialRegister, locals.replaceInput, locals.replaceOutput);
			state.mut().initialRegister1 = locals.replaceInput.newRegister;

			locals.replaceInput.oldRegister = state.get().initialRegister2;
			locals.replaceInput.newRegister = ID(_F, _I, _H, _O, _T, _E, _X, _I, _O, _K, _E, _M, _W, _A, _W, _Q, _J, _T, _O, _P, _Z, _E, _B, _A, _J, _T, _G, _A, _A, _Y, _O, _H, _B, _Q, _F, _W, _M, _X, _V, _B, _A, _D, _Q, _H, _H, _N, _G, _G, _U, _J, _H, _L, _S, _D, _Y, _G);
			CALL(replaceInitialRegister, locals.replaceInput, locals.replaceOutput);
			state.mut().initialRegister2 = locals.replaceInput.newRegister;

			locals.replaceInput.oldRegister = state.get().initialRegister3;
			locals.replaceInput.newRegister = ID(_U, _Z, _S, _T, _A, _D, _S, _M, _J, _D, _W, _K, _Y, _C, _A, _Y, _I, _G, _X, _L, _G, _N, _U, _X, _P, _N, _O, _C, _I, _X, _M, _G, _Q, _I, _U, _L, _G, _Z, _Q, _R, _J, _G, _I, _M, _N, _D, _B, _G, _K, _A, _E, _L, _V, _T, _T, _E);
			CALL(replaceInitialRegister, locals.replaceInput, locals.replaceOutput);
			state.mut().initialRegister3 = locals.replaceInput.newRegister;

			if (state.get().registers.needsCleanup()) { state.mut().registers.cleanup(); }
		}
	}
```

In `INITIALIZE()`, the IDs assigned to `initialRegister1`, `initialRegister2` and `initialRegister3` change to the same three new wallets.
