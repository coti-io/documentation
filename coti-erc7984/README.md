# COTI confidential tokens & ERC-7984 compatibility

**Confidential tokens with encrypted balances and transfers — without treating an unfinished standard as product identity.**

[ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) describes a confidential fungible-token interface aimed at FHE-style stacks (notably Zama / OpenZeppelin confidential contracts). COTI does **not** implement ERC-7984 as a full standard. COTI provides a **confidential-token alternative** with selective compatibility where it helps explorers and integrators — and deliberate divergence where ERC-7984’s design fights COTI’s on-chain privacy model.

This section is a research comparison: why the designs differ, what that unlocks on COTI, and how deployed PoD confidential wrappers fit in.

## Why COTI is not “an ERC-7984 implementation”

| Reason | What it means |
| :----- | :------------ |
| **Draft standard** | ERC-7984 is incomplete; locking product identity to it is premature. |
| **Ciphertext format** | COTI `it*` / `gt*` / `ct*` is not Zama `euint*` / handles. Full drop-in interface compatibility is blocked by crypto, not only API taste. |
| **Allowances, not operators** | ERC-7984 replaces amount-bounded `approve` with time-boxed **unlimited** `setOperator`. COTI keeps encrypted allowances (ERC-20 / ERC-2612-like) because private state lives on-chain and can be compared. |

Compatibility where useful: familiar transfer shapes, metadata, and explorer-facing confidential-transfer signals — **without** claiming full ERC-7984 compliance.

## Comparison at a glance

| Dimension | COTI confidential tokens | Zama / FHE ERC-7984-style |
| :-------- | :----------------------- | :------------------------ |
| Private state | Encrypted values **on-chain** (readable by contracts) | Handles on-chain; bulk ciphertext / execution often off-chain (DAL / coprocessor) |
| Delegation | Encrypted **amount-bounded** `approve` / `transferFrom` | Public **time-boxed operator** (any amount until expiry) |
| User decrypt | Client AES decrypt of on-chain `ct*` | Relayer HTTPS → Gateway/KMS re-encrypt → client |
| Input validity | On-chain `validateCiphertext` (precompile) | Client ZK input proof (`inputProof`), often multi-KB |
| Numeric range | **256-bit** | Typically **64-bit** (`euint64`) |
| Host deployment | Any EVM with a PoD Inbox (computation on COTI) | FHE-capable stack / coprocessor assumptions |

## In this section

1. [**Compatibility and divergence**](compatibility-and-divergence.md) — draft standard, format mismatch, operators vs allowances.
2. [**On-chain data availability**](on-chain-data-availability.md) — private state as contract state; vote → claim public ERC-20.
3. [**Decryption trust model**](decryption-trust-model.md) — client AES vs Relayer/KMS path (TypeScript).
4. [**Input validation**](input-validation.md) — `validateCiphertext` vs browser ZK proofs.
5. [**Precision and decimals**](precision-and-decimals.md) — 256-bit vs 64-bit ceiling.
6. [**Host-chain deployment**](host-chain-deployment.md) — PoD Inbox; token on your chain.
7. [**Transaction economics**](transaction-economics.md) — ciphertext size and on-chain footprint.
8. [**Transfer semantics**](transfer-semantics.md) — encrypted allowances and silent insufficient-balance handling.
9. [**Concurrency**](concurrency.md) — multiple in-flight requests per account.
10. [**Deployed contracts**](deployed-contracts.md) — live pTokens on Fuji and Sepolia.

## Why teams choose COTI’s model

| | |
| :--------------------------- | :------------------------------------------------------------------------ |
| **Wrap real assets**         | 18-decimal confidential WETH and WAVAX — out of reach for 64-bit designs |
| **On-chain private logic**   | Private vars participate in contract control flow alongside public actions |
| **Keep your chain**          | Runs on any EVM chain with a PoD Inbox; no privacy L1 migration           |
| **Full precision**           | 256-bit values, no supply ceiling                                         |
| **Small on-chain footprint** | ~192-byte inputs, 2-slot balances                                         |
| **Instant for users**        | Encrypt and sign — no client-side ZK input proof                          |
| **Private allowances**       | Exact encrypted amounts, not public blanket operator authority            |
| **Local decrypt**            | User `ct*` decrypted with the account AES key on the client               |
