# Compatibility and divergence

COTI offers confidential fungible tokens. It does **not** ship a complete [ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) implementation. This page explains the three reasons that matter for product and research decisions.

## 1. ERC-7984 is not a finished standard

ERC-7984 is a draft interface for confidential fungible tokens. The surface (operators, confidential transfer shapes, encrypted amount pointers) is still evolving in the EIP process and in reference stacks such as [OpenZeppelin confidential contracts](https://docs.openzeppelin.com/confidential-contracts/token).

Treating “ERC-7984-compliant” as product identity is premature: the standard is incomplete, and implementations disagree on crypto formats and trust assumptions. COTI tracks the conversation and remains **selectively compatible** (familiar transfer ideas, metadata, explorer-facing signals) without locking the product to a draft.

## 2. Encrypted data formats differ

ERC-7984 leaves “pointer” resolution to the implementation. In practice, Zama / FHE stacks use encrypted handles (`euint*`, `externalEuint64` + `inputProof`, ACL). COTI uses a different type system:

| Role | COTI | Zama / FHE ERC-7984-style |
| :--- | :--- | :------------------------ |
| User input to a contract | `it*` (ciphertext + signature) | `externalEuint*` + ZK `inputProof` |
| Network / MPC private state | `gt*` (garbled / network-key) | Handle to ciphertext managed with ACL + coprocessor |
| User-readable output | `ct*` (bound to the user’s AES key) | Handle; decrypt via Relayer / KMS path |

Because the ciphertext formats are not interchangeable, a COTI confidential token cannot be a byte-compatible drop-in for a Zama ERC-7984 token. “Compatibility” here means product and integrator familiarity — not shared crypto.

See [Input validation](input-validation.md) for how COTI verifies `it*` on-chain, and [Decryption trust model](decryption-trust-model.md) for how `ct*` is read on the client.

## 3. Operators vs encrypted allowances

ERC-7984 **drops amount-bounded allocation** from the ERC-20 model. Instead of `approve(spender, amount)`, the draft uses:

```solidity
// ERC-7984-style: public relationship, unlimited amount until `until`
function setOperator(address operator, uint48 until) external;
function isOperator(address holder, address spender) external view returns (bool);
```

While approved, an operator may transfer **any amount** on behalf of the holder. The operator relationship and expiry are plaintext on-chain; only balances and transfer amounts stay encrypted. That design reduces FHE compare/branch complexity when encrypted allowance state is hard to keep on-chain — but it is a **dangerous trust model** for users: granting an operator is closer to handing the wallet’s token control to another address until expiry.

COTI keeps **allocation functions similar to ERC-20 and ERC-2612**:

- Exact encrypted amounts via `approve` / `transferFrom` (and related increase/decrease allowance flows where exposed).
- Allowance values stored as ciphertext, readable only by the parties that need them.
- No public “this address may spend everything until time T” grant as the primary delegation model.

That is possible because COTI holds **encrypted state on-chain** and can compare against encrypted allowances inside private execution. It is not a missing ERC-7984 feature — it is a deliberate alternative enabled by [on-chain data availability](on-chain-data-availability.md).

| | COTI | ERC-7984 / Zama-style |
| :- | :--- | :-------------------- |
| Delegation primitive | Encrypted amount-bounded allowance | Time-boxed unlimited operator |
| Amount visibility | Private | Operator may move any amount |
| Relationship visibility | Spender/owner ciphertexts; not a public blanket grant | `OperatorSet` / `isOperator` are public |
| Why | On-chain private state supports compare | Avoid FHE allowance compare / off-chain load |

For transfer behaviour (silent insufficient balance, encrypted approve shape), see [Transfer semantics](transfer-semantics.md). For native COTI and PoD implementations, see the PrivateERC20 / pERC20 developer guides under COTI Privacy Portal and Build on COTI.
