# Transfer semantics

## Allowances vs operators

COTI confidential tokens keep **amount-bounded** `approve` / `transferFrom` (and ERC-2612-style permit where exposed). The allowance is an on-chain ciphertext. See [Compatibility and divergence](compatibility-and-divergence.md) for ERC-7984 `setOperator` (public, unlimited until `until`).

## Insufficient encrypted balance

When the transfer **amount is encrypted**, both stacks avoid reverting on insolvency so observers cannot distinguish “not enough” from a successful private transfer.

| Stack | Mechanism | Host-visible result |
| :---- | :-------- | :------------------ |
| COTI PoD pToken (encrypted amount) | COTI-side `mux` of the effective amount to zero; callback **Success** | Request completes; balances unchanged if insolvent |
| OpenZeppelin `ERC7984` | `FHESafeMath.tryDecrease` + `FHE.select(success, amount, 0)` | Transaction succeeds; `ConfidentialTransfer` amount is an encrypted zero |

COTI **public** amounts (for example portal withdraw where fail vs success must be real) take the other branch: decrypt the comparison and return **Failure** instead of mux-to-zero.

Encrypted-path “success with zero moved” is therefore a **confidentiality** choice on both designs, not a COTI-only behaviour.

Overlapping transfers: [Concurrency](concurrency.md).
