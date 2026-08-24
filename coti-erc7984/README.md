# COTI ERC-7984

**Confidential tokens on the chains you already use.**

Privacy on Demand brings encrypted balances and encrypted transfers to any EVM chain — no privacy-native L1, no specialised rollup, no migration. Live today on Avalanche Fuji and Ethereum Sepolia.

COTI's implementation of the [ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) confidential-token interface is the PoD ERC-7984: a 1:1 collateralised confidential wrapper deployed on the host chain, with the encrypted computation performed on COTI behind the scenes. It exposes ERC-7984 metadata and emits `ConfidentialTransfer` events, so explorers and integrators treat it as a first-class confidential token.

This section compares that implementation against FHE-based implementations of the same standard, dimension by dimension.

## In this section

* [**Precision and decimals**](precision-and-decimals.md) — 256-bit values against the 64-bit ceiling, and why it decides whether a wrapper can mirror an 18-decimal asset.
* [**Host-chain deployment**](host-chain-deployment.md) — what the token requires of the chain it runs on, and what it requires of the user.
* [**Transaction economics**](transaction-economics.md) — encrypted-input size, on-chain footprint, and where the cryptographic work happens.
* [**Transfer semantics**](transfer-semantics.md) — encrypted allowances with standard `approve` / `transferFrom`, and failures that reveal nothing.
* [**Concurrency**](concurrency.md) — multiple in-flight requests per account, ordered by a monotonic nonce.
* [**Deployed contracts**](deployed-contracts.md) — the six pTokens live on Avalanche Fuji and Ethereum Sepolia, and how new ones are listed.

## Why teams choose COTI ERC-7984

|                              |                                                                           |
| :--------------------------- | :------------------------------------------------------------------------ |
| **Wrap real assets**         | 18-decimal confidential WETH and WAVAX — out of reach for 64-bit designs |
| **Keep your chain**          | Runs on any EVM chain; no privacy L1, no migration                        |
| **Full precision**           | 256-bit values, no supply ceiling                                         |
| **Small on-chain footprint** | ~192-byte inputs, 2-slot balances                                         |
| **Instant for users**        | No client-side proof generation                                           |
| **Private allowances**       | Exact encrypted amounts, not public blanket authority                     |
| **Ship fast**                | Factory-deployed portal + token pair per asset                            |
