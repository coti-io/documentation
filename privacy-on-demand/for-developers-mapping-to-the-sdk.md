# For developers: mapping concepts to the SDK

This page is the **bridge** from [Architecture and main components](architecture-and-components.md) to the canonical [PoD SDK documentation on GitHub](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs). It repeats a few facts on purpose so engineers can verify mental models quickly.

For a guided first implementation, read **[Tutorials: building Privacy on Demand (PoD) dApps](tutorials-privacy-on-demand.md)** to pick the integration model, then follow [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) for a **primitive-only** Solidity + TypeScript walkthrough (Sepolia presets).

## Official reading order (SDK)

The upstream docs recommend:

1. [Privacy dApps on any EVM chain with COTI PoD](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/01-privacy-decentralized-apps-on-any-evm-chain-with-coti-pod.md)
2. [Getting started](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/04-getting-started.md)
3. [Writing privacy contracts on Ethereum](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05-writing-privacy-contracts-on-ethereum.md)
4. [TypeScript integration (UX development)](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/06-typescript-integration-ux-development.md)

Then deep dives:

- [Async execution](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05a-async-execution.md)
- [MPC library (PodLib)](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05b-multi-party-computing-library-mpclib.md)
- [Examples with description](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05c-examples-with-description.md)
- Contract references: [Data types](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/01-it-ct-gt-data-types.md), [Patterns and checklist](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/02-contract-patterns-and-checklist.md), [Request builder and remote calls](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/03-request-builder-and-remote-calls.md), [Fees, gas, and oracle](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md)

## Component → source file map

| Concept (this book) | Where it lives in the SDK docs / repo |
| --- | --- |
| **Inbox** | [IInbox.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/IInbox.sol) and cross-domain flow in the [domain model](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/01-privacy-decentralized-apps-on-any-evm-chain-with-coti-pod.md) diagram. |
| **Callback guard** | [InboxUser.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/InboxUser.sol) (`onlyInbox`) — see [Features](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/03-features.md). |
| **PodLib** | [PodLib.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/mpc/PodLib.sol) and width-specific libraries (`PodLib64`, `PodLib128`, `PodLib256`). |
| **PodUser / presets** | [PodUser.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/mpc/PodUser.sol), network mixins such as [PodUserSepolia.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/mpc/PodUserSepolia.sol) in [Getting started](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/04-getting-started.md). |
| **Types (`it*`, `ct*`, `gt*`)** | [MpcCore.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/utils/mpc/MpcCore.sol) and [Data types](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/01-it-ct-gt-data-types.md). |
| **Custom COTI calls** | [MpcAbiCodec.sol](https://github.com/cotitech-io/coti-pod-sdk/blob/main/contracts/mpccodec/MpcAbiCodec.sol) and the **custom mode** section of [Writing privacy contracts](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05-writing-privacy-contracts-on-ethereum.md). |
| **Client crypto** | [coti-pod-crypto.ts](https://github.com/cotitech-io/coti-pod-sdk/blob/main/src/coti-pod-crypto.ts) via `CotiPodCrypto` ([TypeScript integration](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/06-typescript-integration-ux-development.md)). |

## Implementation checklist (condensed)

Derived from the SDK’s [Writing privacy contracts](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05-writing-privacy-contracts-on-ethereum.md) and [Async execution](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05a-async-execution.md):

1. **Classify data** — public metadata vs `it*` inputs vs `ct*` outputs vs internal `gt*` (COTI-only).
2. **Pick integration mode** — `PodLib` helpers vs custom `MpcAbiCodec` + COTI contract.
3. **Model async state** — persist `requestId`, track pending/completed/failed.
4. **Harden callbacks** — `onlyInbox`, correct `abi.decode` tuple, validate peer context when applicable.
5. **Configure routing safely** — gated `configure` / `configureCoti` / inbox updates.
6. **Budget fees** — understand `msg.value` and `callbackFeeLocalWei`; use Inbox fee views where available ([Fees doc](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md)).
7. **Test failure paths** — spoofed callback must revert, error callbacks must mark failures, decrypt integration must match widths.

## Relationship to native COTI “build” documentation

If you build **directly on COTI V2** with precompiles and private types, start from **[Build on COTI](../build-on-coti/README.md)**. PoD adds the **Inbox-mediated cross-chain** angle; many **cryptographic ideas rhyme**, but **deployment and UX** differ.

## Package install

```bash
npm install "@coti/pod-sdk"
```

See [Getting started](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/04-getting-started.md) for Solidity imports and the `contract MyApp is PodLib, PodUserSepolia` pattern.
