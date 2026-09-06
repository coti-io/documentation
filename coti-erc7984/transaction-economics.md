# Transaction economics

Confidentiality has a size and a **path** cost. On Zama FHEVM the client ZKPoK is sent to the Relayer. The host chain receives a short coprocessor attestation, not that proof.

Measured **6 September 2026** (`@zama-fhe/sdk` 3.5.1, Sepolia cUSDCMock):

| | **COTI confidential tokens** | **Zama / FHE ERC-7984-style** |
| :--------------- | :--------------------------- | :---------------------------- |
| Encrypted input on the host chain | **`itUint256` ~192-byte payload** (two 32-byte limbs + ~65-byte signature) | 32-byte handle + **230-byte** `inputProof` (1 handle, 3 signers) |
| Typical `confidentialTransfer` calldata | Same order of magnitude as a signed `it*` transfer | **452 bytes** in 8 of 9 sampled txs |
| Packed ciphertext + ZKPoK | Not used | **18,794 bytes** in Relayer `ciphertextWithInputVerification` |
| Where validity is checked | `validateCiphertext` in the same host/COTI call | Relayer / coprocessors first; `InputVerifier` signatures on-chain |
| Client-side work | Encrypt and sign (WASM ZK proving not required) | Generate ZKPoK (WASM), wait for Relayer |
| Numeric type in the token reference | 256-bit | `euint64` + 6 decimals in OpenZeppelin `ERC7984` |

On-chain encrypted-amount size is hundreds of bytes on both stacks. What differs:

- Users pay for **ZKPoK verification** on the Gateway path even though the ZK proof never appears in host calldata.
- Input registration and decryption **depend on the Relayer**. A Relayer or coprocessor failure aborts **before** the host transaction exists.
- Host `confidentialTransfer` gas on the sampled Sepolia txs was about **0.96M–1.85M**.
- FHE arithmetic is metered in [HCU](https://docs.zama.org/protocol/solidity-guides/development-guide/hcu) on the coprocessor, separate from calldata.

COTI balances that are `ctUint256` occupy two ciphertext limbs (see [On-chain data availability](on-chain-data-availability.md)). Zama balances are 32-byte handles; the ciphertext lives with the coprocessors.

For the Relayer flow, `InputVerifier` layout, and Solidity `validateCiphertext` examples, see [Input validation](input-validation.md).
