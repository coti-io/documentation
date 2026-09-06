# Compatibility and divergence

COTI offers confidential fungible tokens. It does **not** ship a complete [ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) implementation. ERC-7984 is a **Draft** ERC (created 3 July 2025; citation still `[DRAFT]` as of 6 September 2026).

## 1. Draft interface, multiple cryptosystems

ERC-7984 standardises names and `bytes32` **pointers**. Pointer resolution, ciphertext format, and trust assumptions are implementation-specific. The common reference is [OpenZeppelin confidential contracts](https://docs.openzeppelin.com/confidential-contracts/token) on Zama FHEVM (`@openzeppelin/confidential-contracts` 0.5.3, balances as `euint64`).

COTI uses a different crypto (`it*` / `gt*` / `ct*`). Selective compatibility (transfer shapes, metadata, explorer-facing confidential-transfer signals) is not byte-compatible ERC-7984.

## 2. Encrypted data formats

| Role | COTI | Zama / FHE ERC-7984-style (pinned stack) |
| :--- | :--- | :---------------------------------------- |
| User input | `it*` (ciphertext + signature) | `externalEuint*` handle + `inputProof` (**coprocessor signatures**, not the ZKPoK — [Input validation](input-validation.md)) |
| Network private state | `gt*` (network-key / garbled) | Handle; ciphertext with coprocessors; ACL |
| User-readable output | `ct*` (account AES key) | Handle; decrypt via Relayer / KMS ([Decryption trust model](decryption-trust-model.md)) |

## 3. Operators vs encrypted allowances

ERC-7984 replaces amount-bounded `approve` with:

```solidity
// Public relationship. Operator may transfer any amount until `until`.
function setOperator(address operator, uint48 until) external;
function isOperator(address holder, address spender) external view returns (bool);
```

While the grant is live, the operator can move **any** amount of the holder’s token. `OperatorSet` / `isOperator` are **plaintext**. Balances and transfer amounts stay encrypted. The EIP rationale is to avoid tracking encrypted approval amounts on the external system.

COTI keeps **encrypted, amount-bounded** allowances (ERC-20 / ERC-2612-like `approve` / `transferFrom`):

- Allowance values are ciphertext.
- Compare-and-subtract runs in private execution ([on-chain data availability](on-chain-data-availability.md)).
- There is no public “unlimited until time T” grant as the primary delegation model.

| | COTI | ERC-7984 / OpenZeppelin `ERC7984` |
| :- | :--- | :-------------------------------- |
| Delegation | Encrypted amount-bounded allowance | Time-boxed operator |
| Amount the delegate may spend | The approved encrypted amount | Any amount until `until` |
| Relationship on-chain | Allowance ciphertext | Public `isOperator` / `OperatorSet` |

[Transfer semantics](transfer-semantics.md) covers silent insufficient-balance handling. Native PrivateERC20 and PoD pERC20 developer guides live under COTI Privacy Portal and Build on COTI.
