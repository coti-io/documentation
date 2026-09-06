# COTI confidential tokens & ERC-7984 compatibility

[ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) is a **Draft** ERC (created 3 July 2025) for confidential fungible tokens via `bytes32` pointers. The interface is technology-agnostic; the widely used reference stack is Zama FHEVM plus [OpenZeppelin confidential contracts](https://docs.openzeppelin.com/confidential-contracts/token). COTI does **not** implement ERC-7984 as a complete standard. It offers a confidential-token alternative with overlapping transfer shapes where that helps explorers and integrators, and a different ciphertext format, allowance model, and (on native COTI) decrypt path.

This section compares ciphertext formats, input registration, decrypt paths, numeric range, and host-chain deployment.

## Sources and versions

Compared **6 September 2026** against:

| Item | Version / status |
| :--- | :--------------- |
| ERC-7984 | Draft ([EIP-7984](https://eips.ethereum.org/EIPS/eip-7984)) |
| `@zama-fhe/sdk` | 3.5.1 (depends on `@fhevm/sdk` 0.13.2; TFHE WASM v1.6.2) |
| `@fhevm/solidity` | 0.13.3 |
| `@openzeppelin/confidential-contracts` | 0.5.3 (`ERC7984` stores `euint64`; `decimals()` returns `6`) |
| Zama Sepolia relayer | `https://relayer.testnet.zama.org` |
| Input-size measurement | Relayer `POST /v2/input-proof` plus live `confidentialTransfer` calldata on [cUSDCMock](https://sepolia.etherscan.io/address/0x7c5BF43B851c1dff1a4feE8dB225b87f2C223639) |

Zama figures below are for that stack. Protocol details can change; re-check the pinned packages before treating a number as current.

## Why COTI is not “an ERC-7984 implementation”

| Reason | What it means |
| :----- | :------------ |
| **Draft standard** | ERC-7984 is still a Draft ERC. Implementations already disagree on crypto formats and trust assumptions. |
| **Ciphertext format** | COTI `it*` / `gt*` / `ct*` is not Zama `euint*` / handles. Drop-in interface compatibility is blocked by crypto, not only API taste. |
| **Allowances, not operators** | ERC-7984 replaces amount-bounded `approve` with time-boxed **unlimited** `setOperator`. COTI keeps encrypted allowances (ERC-20 / ERC-2612-like) because private state lives on-chain and can be compared. |

Compatibility where useful: familiar transfer shapes, metadata, and explorer-facing confidential-transfer signals — **without** claiming full ERC-7984 compliance.

## Comparison

| Dimension | COTI confidential tokens | Zama / FHE ERC-7984-style (pinned stack) |
| :-------- | :----------------------- | :---------------------------------------- |
| Private state | Encrypted values **on-chain** (`gt*` / user `ct*`) | 32-byte handles on-chain; ciphertext held by coprocessors |
| Delegation | Encrypted **amount-bounded** `approve` / `transferFrom` | Public **time-boxed operator** (any amount until `until`) |
| User decrypt | Client AES decrypt of on-chain `ct*` | Relayer HTTPS → Gateway / threshold KMS re-encrypt → client |
| Contract decrypt → public logic | Native COTI: [synchronous decrypt](../how-coti-works/advanced-topics/coti-vs-others.md#advantages-over-fhe) in the same call. PoD host-chain apps: [async request/callback](../privacy-on-demand/async-private-operations.md) | Encrypted branching via `FHE.select` in the same tx. Public branching needs a **later** tx with KMS signatures ([Zama branching](https://docs.zama.org/protocol/solidity-guides/smart-contract/logics/conditions)) |
| Input validity | On-chain `validateCiphertext` (precompile `0x64`) | Client ZKPoK sent to the Relayer; coprocessors verify; host chain checks **signed handles** |
| Typical encrypted input on the host chain | `itUint256`: two 32-byte limbs + ~65-byte signature (**~192 bytes** payload) | `externalEuint64` handle (32 bytes) + `inputProof` attestation (**230 bytes** for 1 handle / 3 signers) |
| Off-chain input blob | None for native encrypt; PoD HTTP encrypt is a separate helper | Packed ciphertext + ZKPoK to Relayer: **18,794 bytes** in this measurement |
| Numeric type in the token reference | **256-bit** (`itUint256` / `gtUint256`) | **`euint64`** in OpenZeppelin `ERC7984`; `decimals()` = **6**. `euint128` has the same arithmetic ops at higher [HCU](https://docs.zama.org/protocol/solidity-guides/development-guide/hcu); `euint256` has **no** add/mul |
| Trust | MPC / garbled-circuit network (threshold network key); user AES key on the client; PoD encryption HTTP helper **does** receive plaintext | Threshold coprocessors (InputVerifier signatures); threshold KMS; Relayer is a liveness path (documented as untrusted for plaintext) |
| Host deployment | Any EVM with a PoD Inbox (computation on COTI) | FHEVM host contracts + coprocessor / Gateway assumptions |
| Concurrency | PoD: multiple in-flight requests per account, applied by request id / nonce | Host txs follow ordinary EVM account nonces; overlapping writes to the same handle are ordered by coprocessor execution of host events. Unwrap is two host txs |

## In this section

1. [**Compatibility and divergence**](compatibility-and-divergence.md) — draft standard, format mismatch, operators vs allowances.
2. [**On-chain data availability**](on-chain-data-availability.md) — private state as contract state; private → public control flow.
3. [**Decryption trust model**](decryption-trust-model.md) — client AES vs Relayer/KMS; encryption helper plaintext.
4. [**Input validation**](input-validation.md) — `validateCiphertext` vs Relayer ZKPoK + on-chain signatures.
5. [**Precision and decimals**](precision-and-decimals.md) — 256-bit vs `euint64` / 6 decimals / `euint128` cost.
6. [**Host-chain deployment**](host-chain-deployment.md) — PoD Inbox; token on your chain.
7. [**Transaction economics**](transaction-economics.md) — measured calldata vs off-chain proof size.
8. [**Transfer semantics**](transfer-semantics.md) — encrypted allowances and silent insufficient-balance handling.
9. [**Concurrency**](concurrency.md) — in-flight PoD requests vs Zama handle updates.
10. [**Deployed contracts**](deployed-contracts.md) — live pTokens on Fuji and Sepolia.
