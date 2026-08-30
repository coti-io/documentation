# Transaction economics

## Built for real transaction economics

Confidentiality usually arrives with a size problem. Zama / FHE confidential tokens attach a zero-knowledge **input proof** to every encrypted value — commonly **16–20 KB per input** — and often keep multi-kilobyte ciphertext off-chain behind handles.

COTI encrypted inputs are small enough to treat like ordinary transaction data. Validity is checked on-chain with [`validateCiphertext`](input-validation.md), not proven in the browser.

|                  | **COTI confidential tokens** | **Zama / FHE ERC-7984-style**            |
| :--------------- | :--------------------------- | :--------------------------------------- |
| Encrypted input  | **~192 bytes**               | ~16,000–20,000 bytes (with `inputProof`) |
| On-chain balance | **2 storage slots**          | Handle on-chain, multi-KB blob offloaded |
| Client-side work | **Encrypt and sign**         | Generate a ZK proof, per transaction     |
| Numeric range    | **256-bit**                  | 64-bit                                   |

**Roughly two orders of magnitude smaller on input.** Confidential transfers that fit comfortably inside normal block economics, on chains that were never designed for privacy.

For proving cost, WASM/mobile constraints, and Solidity `validateCiphertext` examples, see [Input validation](input-validation.md).
