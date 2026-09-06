# For developers: mapping concepts to the SDK

This page maps [Architecture and main components](architecture-and-components.md) to the packages you install:

- TypeScript: [`@coti-io/pod-sdk`](https://www.npmjs.com/package/@coti-io/pod-sdk) ([GitHub](https://github.com/coti-io/coti-sdk-pod))
- Solidity: [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts) (`contracts/pod/`)

For a guided first implementation, read **[Tutorials: building Privacy on Demand (PoD) dApps](tutorials-privacy-on-demand.md)**, then [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md).

## Reading order

1. [What is Privacy on Demand?](what-is-privacy-on-demand.md)
2. [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md)
3. [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md)
4. [TypeScript PoD SDK](typescript-pod-sdk.md)

Then:

- [Async private operations](async-private-operations.md)
- [Architecture and main components](architecture-and-components.md) (PodLib, types)
- [Tutorials index](tutorials-privacy-on-demand.md)
- [How a private request travels end to end](how-a-private-request-travels-end-to-end.md)
- [How do PoA fees work?](how-poa-fees-work.md)

## Component → source file map

| Concept (this book) | Where it lives |
| --- | --- |
| **Inbox** | [IInbox.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/IInbox.sol) |
| **Callback guard** | [InboxUser.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/InboxUser.sol) (`onlyInbox`) |
| **PodLib** | [PodLib.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib.sol) and width-specific libraries (`PodLib64`, `PodLib128`, `PodLib256`) |
| **PodUser / presets** | [PodUser.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUser.sol), [PodUserSepolia.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserSepolia.sol) |
| **Types (`it*`, `ct*`, `gt*`)** | [MpcCore.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/utils/mpc/MpcCore.sol) — see also [data shapes](architecture-and-components.md) |
| **Custom COTI calls** | [MpcAbiCodec.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpccodec/MpcAbiCodec.sol) and [custom tutorial](tutorial-custom-logic.md) |
| **Client crypto** | [coti-pod-crypto.ts](https://github.com/coti-io/coti-sdk-pod/blob/main/src/coti-pod-crypto.ts) (`CotiPodCrypto`) |

## Implementation checklist

1. **Classify data** — public metadata vs `it*` inputs vs `ct*` outputs vs internal `gt*` (COTI-only).
2. **Pick integration mode** — `PodLib` helpers vs custom `MpcAbiCodec` + COTI contract.
3. **Model async state** — persist `requestId`, track pending/completed/failed.
4. **Harden callbacks** — `onlyInbox`, correct `abi.decode` tuple, validate peer context when applicable.
5. **Configure routing safely** — gated `configure` / `configureCoti` / inbox updates.
6. **Budget fees** — `msg.value` and `callbackFeeLocalWei`; Inbox fee views ([How do PoA fees work?](how-poa-fees-work.md)).
7. **Test failure paths** — spoofed callback must revert, error callbacks must mark failures, decrypt widths must match.

## Relationship to native COTI “build” documentation

If you build **directly on COTI V2** with precompiles and private types, start from **[Build on COTI](../build-on-coti/README.md)**. PoD adds the Inbox-mediated cross-chain path; cryptographic types rhyme, but deployment and UX differ.

## Package install

```bash
npm install @coti-io/pod-sdk ethers
npm install github:coti-io/coti-contracts#main
```

Solidity imports use `@coti-io/coti-contracts/contracts/pod/mpc/...`. The npm TypeScript package does not ship contract sources.
