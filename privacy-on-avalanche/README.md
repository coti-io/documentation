# Privacy on Avalanche (PoD on Fuji)

**Privacy on Avalanche** applies Privacy on Demand to **Avalanche Fuji C-Chain** (chain ID `43113`): keep wallets, assets, and dApp contracts on Avalanche, and send **encrypted private computation** to **COTI Testnet**.

Fees on the host side are paid in **AVAX**. Private execution still happens on COTI; results return as ciphertext to your Fuji contracts for client-side decryption.

> **Development status:** This Privacy on Avalanche material and the **COTI PoD SDK** it describes are **under active development**. Treat them accordingly: on-chain and client code **may not yet be fully audited**, and **breaking changes** (APIs, ABIs, addresses, presets, or documentation) can occur as the stack matures. Pin versions, follow release notes, and perform your own review before relying on anything in production.

<div style="font-size: 1.45rem; line-height: 1.72; margin-bottom: 2rem;">

<h2 style="font-size: 2.35rem; font-weight: 600; margin-top: 0; margin-bottom: 1rem; line-height: 1.2;">Quick Access</h2>

- **[Getting started on Avalanche Fuji (Day 0)](getting-started-fuji.md)** — Wallet, faucet, Hardhat/Foundry, SnowScan smoke-test.
- **[Tutorials: PoD dApps on Avalanche (choose your integration model)](tutorials-privacy-on-avalanche.md)** — Primitive-only vs custom COTI logic, then links to step-by-step Fuji guides.
- **[Architecture and design](architecture-and-components.md)** — Inbox, MPC executor, PodUser, PodLib, and how they connect.
- **[Avalanche Fuji network & addresses](networks/fuji.md)** — Fuji C-Chain RPC, SnowScan, Inbox, Privacy Portal, and sample MpcAdder.
- **[Networks](networks/README.md)** — Fuji (host) + COTI Testnet (private execution) parameters.
- **[Interactive PoD architecture on Fuji (pod.coti.io/avalanche)](https://pod.coti.io/avalanche/index.html)** — Live demo: MpcAdder journey across Avalanche Fuji, relayer, and COTI.
- **[Learn about fees](how-poa-fees-work.md)** — How PoA/PoD fees split across COTI and Fuji (paid in AVAX).
- **[Millionaires demo](https://millionaire.demo.coti.io)** — Live demo (includes an Avalanche Fuji route).

<h2 style="font-size: 2.35rem; font-weight: 600; margin-top: 1.75rem; margin-bottom: 1rem; line-height: 1.2;">Further resources</h2>

- **[Examples](https://github.com/coti-io/coti-contracts/tree/main/contracts/pod/examples)** — Contract examples in `@coti-io/coti-contracts`.
- **[PoD SDK documentation](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs)** — Full SDK docs on GitHub.
- **[General Privacy on Demand section](../privacy-on-demand/README.md)** — Multi-host PoD overview (not Avalanche-only).

</div>

---

This section is the **Avalanche Fuji–first** companion to Privacy on Demand: same PoD pattern, with Fuji C-Chain network constants, `PodUserFuji` presets, SnowScan explorers, and AVAX fee context. For integration, use [`@coti-io/pod-sdk`](https://www.npmjs.com/package/@coti-io/pod-sdk) (TypeScript) and [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts) (Solidity), plus the links below.

## Fuji at a glance

| Parameter | Value |
| --- | --- |
| Network | Avalanche Fuji C-Chain |
| Chain ID | `43113` |
| Native token | AVAX |
| RPC | `https://api.avax-test.network/ext/bc/C/rpc` |
| Explorer | [testnet.snowscan.xyz](https://testnet.snowscan.xyz) |
| Inbox | `0xAb625bE229F603f6BBF964474AFf6d5487e364De` |
| Private execution | COTI Testnet (`7082400`) |
| Solidity preset | `PodUserFuji` |

Full tables: [Avalanche Fuji](networks/fuji.md).

## Who this documentation is for

| Audience | What you will get here |
| --- | --- |
| **Product, compliance, and business readers** | Plain-language model of privacy on Avalanche + COTI, where data lives, and what “async” private operations mean in practice. |
| **Architects and technical leads** | End-to-end diagrams for the **Fuji ↔ COTI** path, component roles, and domain boundaries. |
| **Developers** | Fuji-focused map from concepts to Solidity/TypeScript (`PodUserFuji`, AVAX fees, SnowScan), plus pointers to the authoritative SDK docs. |

## Table of contents

### Start here

1. [Getting started on Avalanche Fuji (Day 0)](getting-started-fuji.md) — Wallet, faucet, Hardhat/Foundry, SnowScan.

### Understand first (readable without writing code)

2. [What is Privacy on Avalanche?](what-is-privacy-on-avalanche.md) — Problem, promise, and constraints with Fuji as the host chain.
3. [How a private request travels end to end](how-a-private-request-travels-end-to-end.md) — Timeline from user action on Fuji to decrypted result.
4. [Architecture and main components](architecture-and-components.md) — Where **Inbox**, **MPC executor**, **PodUser**, and **PodLib** sit, with diagrams.
5. [Networks](networks/README.md) — [Avalanche Fuji](networks/fuji.md) (host) and [COTI Testnet](networks/coti-testnet.md) (execution).
6. [Glossary](glossary.md) — Short definitions of terms you will see in PoD and SDK docs.

### Deeper context

7. [Async private operations (why it is not instant)](async-private-operations.md) — What “pending” means and why UX must reflect it.
8. [How do PoA fees work?](how-poa-fees-work.md) — Two-way Inbox budgets in AVAX, oracle conversion, and a worked gas-unit example.
9. [For developers: mapping concepts to the SDK](for-developers-mapping-to-the-sdk.md) — Checklists and links to the [PoD SDK documentation on GitHub](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs).

### Tutorials (hands-on)

10. [Tutorials: building PoD dApps on Avalanche](tutorials-privacy-on-avalanche.md) — When to use **MpcLib / PodLib** primitives vs **custom COTI + Fuji** contracts.
11. [TypeScript PoD SDK (`CotiPodCrypto`, `PodContract`)](typescript-pod-sdk.md) — Encryption/decryption, fee estimation, method calls, and request ID extraction.
12. [Cookbook: private investor allocations with PoD](cookbook-private-investor-allocations.md) — Start from a familiar public Fuji allocation dApp, then make allocation reads and withdrawals private with PoD.
13. [Tutorial: private Adder on Avalanche Fuji](tutorial-private-adder-fuji.md) — Minimal primitive-only adder: `PodUserFuji`, AVAX fees, TypeScript crypto.
14. [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md) — Encrypted messaging shape: `DirectMessageCotiSide` + Fuji orchestrator.

## Official technical reference

The machine-readable contracts, types, and APIs live in the open-source SDK. Treat this book chapter as the **human-oriented companion**; treat the repository as the **source of truth** for signatures, fees, and network constants:

- [COTI PoD SDK — documentation index](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs)
