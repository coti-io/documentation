# For developers: mapping concepts to the SDK

This page is the **bridge** from [Architecture and main components](architecture-and-components.md) to the canonical [PoD SDK documentation on GitHub](https://github.com/coti-io/coti-sdk-pod/tree/main/site). It repeats a few facts on purpose so engineers can verify mental models quickly.

For a guided first implementation, read **[Tutorials: building Privacy on Demand (PoD) dApps](tutorials-privacy-on-demand.md)** to pick the integration model, then follow [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) for a **primitive-only** Solidity + TypeScript walkthrough (Sepolia presets).

## Official reading order (SDK)

The upstream docs recommend:

1. [Privacy dApps on any EVM chain with COTI PoD](https://github.com/coti-io/coti-sdk-pod/tree/main/site/01-privacy-decentralized-apps-on-any-evm-chain-with-coti-pod)
2. [Getting started](https://github.com/coti-io/coti-sdk-pod/tree/main/site/04-getting-started)
3. [Writing privacy contracts on Ethereum](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05-writing-privacy-contracts-on-ethereum)
4. [TypeScript integration (UX development)](https://github.com/coti-io/coti-sdk-pod/tree/main/site/06-typescript-integration-ux-development)

Then deep dives:

- [Async execution](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05a-async-execution)
- [MPC library (PodLib)](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05b-multi-party-computing-library-mpclib)
- [Examples with description](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05c-examples-with-description)
- Contract references: [Data types](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/01-it-ct-gt-data-types), [Patterns and checklist](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/02-contract-patterns-and-checklist), [Request builder and remote calls](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/03-request-builder-and-remote-calls), [Fees, gas, and oracle](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/04-fees-gas-and-oracle)

## Component → source file map

| Concept (this book) | Where it lives |
| --- | --- |
| **Inbox** | [IInbox.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/IInbox.sol) and cross-domain flow in the [domain model](https://github.com/coti-io/coti-sdk-pod/tree/main/site/01-privacy-decentralized-apps-on-any-evm-chain-with-coti-pod) diagram. |
| **Callback guard** | [InboxUser.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/InboxUser.sol) (`onlyInbox`) — see [Features](https://github.com/coti-io/coti-sdk-pod/tree/main/site/03-features). |
| **PodLib** | [PodLib.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib.sol) and width-specific libraries (`PodLib64`, `PodLib128`, `PodLib256`). |
| **PodUser / presets** | [PodUser.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUser.sol), network mixins such as [PodUserSepolia.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserSepolia.sol). |
| **Types (`it*`, `ct*`, `gt*`)** | [MpcCore.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/utils/mpc/MpcCore.sol) in **`@coti-io/coti-contracts`** (not published inside the npm SDK). See [Data types](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/01-it-ct-gt-data-types). |
| **Custom COTI calls** | [MpcAbiCodec.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpccodec/MpcAbiCodec.sol) and the **custom mode** section of [Writing privacy contracts](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05-writing-privacy-contracts-on-ethereum). |
| **Client crypto** | [coti-pod-crypto.ts](https://github.com/coti-io/coti-sdk-pod/blob/main/src/coti-pod-crypto.ts) via `CotiPodCrypto` ([TypeScript integration](https://github.com/coti-io/coti-sdk-pod/tree/main/site/06-typescript-integration-ux-development)). |

## Type model at a glance

`gt*` / `ct*` / `it*` types live in **`@coti-io/coti-contracts`** [`MpcCore.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/utils/mpc/MpcCore.sol). The npm package **`@coti-io/pod-sdk` publishes TypeScript only** (`dist`) — it does **not** ship Solidity or a vendored `MpcCore`.

In the current revision:

- **`gtUint8` … `gtUint256` and `gtBool`** are **user‑defined value types** (`type gtUint256 is uint256`). Pass and assign them like `uint256` — **no `memory` / `calldata` on `gt*` parameters or locals**.
- **`ctUint8` … `ctUint128`** are also user‑defined value types (single `uint256` word).
- **`ctUint256`** is a **struct** `{ ctUint128 ciphertextHigh; ctUint128 ciphertextLow; }` — decoded locals and callback variables must use a `memory` location, and off‑chain reads return the two limbs as a tuple.
- **`itUint*`** (user encrypted inputs, `ciphertext + signature`), **`utUint*`** (dual‑ciphertext), **`gtString`** and **`ctString`** remain structs — keep their `calldata` / `memory` locations.

Import Solidity types from **`@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol`**. Off‑chain decryption uses **`@coti-io/coti-sdk-typescript@^1.0.7`**, which exposes `decryptUint256({ ciphertextHigh, ciphertextLow }, accountAesKey)` for the 256‑bit lane (also wrapped by `CotiPodCrypto.decrypt` in `@coti-io/pod-sdk`).

## Implementation checklist (condensed)

Derived from the SDK’s [Writing privacy contracts](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05-writing-privacy-contracts-on-ethereum) and [Async execution](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05a-async-execution):

1. **Classify data** — public metadata vs `it*` inputs vs `ct*` outputs vs internal `gt*` (COTI-only).
2. **Pick integration mode** — `PodLib` helpers vs custom `MpcAbiCodec` + COTI contract.
3. **Model async state** — persist `requestId`, track pending/completed/failed.
4. **Harden callbacks** — `onlyInbox`, correct `abi.decode` tuple, validate peer context when applicable.
5. **Configure routing safely** — gated `configure` / `configureCoti` / inbox updates.
6. **Budget fees** — understand `msg.value` and `callbackFeeLocalWei`; use Inbox fee views where available ([Fees doc](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/04-fees-gas-and-oracle)).
7. **Test failure paths** — spoofed callback must revert, error callbacks must mark failures, decrypt integration must match widths.

## Relationship to native COTI “build” documentation

If you build **directly on COTI V2** with precompiles and private types, start from **[Build on COTI](../build-on-coti/README.md)**. PoD adds the **Inbox-mediated cross-chain** angle; many **cryptographic ideas rhyme**, but **deployment and UX** differ.

Do not confuse the **PoD cross-chain Privacy Portal** (Fuji/Sepolia host pTokens) with the **[native COTI Privacy Portal](../coti-privacy-portal/README.md)** (`PrivateERC20` on COTI).

## Package install

```bash
npm install @coti-io/pod-sdk ethers
npm install github:coti-io/coti-contracts#main
```

- **TypeScript:** `@coti-io/pod-sdk`
- **Solidity:** `@coti-io/coti-contracts` (`PodLib`, `PodUserSepolia`, `MpcCore`, …) — the npm SDK does not ship contract sources

Use `contract MyApp is PodLib, PodUserSepolia` with imports from `@coti-io/coti-contracts/contracts/pod/mpc/...`.
