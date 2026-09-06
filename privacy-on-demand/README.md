# Privacy on Demand (PoD)

Privacy on Demand lets applications use **strong privacy for data and computation** while still using **ordinary EVM chains** (Ethereum, L2s, and other compatible networks) for accounts, assets, and business workflows.

> **Development status:** This Privacy on Demand material and the **COTI PoD SDK** it describes are **under active development**. Treat them accordingly: on-chain and client code **may not yet be fully audited**, and **breaking changes** (APIs, ABIs, addresses, presets, or documentation) can occur as the stack matures. Pin versions, follow release notes, and perform your own review before relying on anything in production.

<div style="font-size: 1.45rem; line-height: 1.72; margin-bottom: 2rem;">

<h2 style="font-size: 2.35rem; font-weight: 600; margin-top: 0; margin-bottom: 1rem; line-height: 1.2;">Quick Access</h2>

- **[Tutorials: PoD dApps (choose your integration model)](tutorials-privacy-on-demand.md)** — Primitive-only vs custom COTI logic, then links to step-by-step guides.
- **[Architecture and design](architecture-and-components.md)** — Inbox, MPC executor, PodUser, PodLib, and how they connect.
- **[Networks](networks/README.md)** — COTI Testnet and Avalanche Fuji parameters plus PoD contract addresses.
- **[Interactive PoD architecture (pod.coti.io)](https://pod.coti.io/)** — Live demo: play the MpcAdder journey across Sepolia, relayer, and COTI, with GitHub source links and gas/fee visualization.
- **[Learn about fees](how-poa-fees-work.md)** — How PoA/PoD fees split across COTI and your host chain.
- **[Millionaires demo](https://millionaire.demo.coti.io)** — Live demo (external).

<h2 style="font-size: 2.35rem; font-weight: 600; margin-top: 1.75rem; margin-bottom: 1rem; line-height: 1.2;">Further resources</h2>

- **[Examples](https://github.com/cotitech-io/coti-pod-sdk/tree/main/contracts/examples)** — Contract examples in the PoD SDK repo.
- **[PoD SDK documentation](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs)** — Full SDK docs on GitHub.

The same **Quick Access** and **Further resources** blocks appear on the [docs homepage](../README.md).

</div>

---

This section explains **what PoD is**, **how it feels to users and operators**, and **how the main pieces fit together**. For step-by-step integration with the **COTI PoD SDK**, use the [npm package](https://www.npmjs.com/package/@coti/pod-sdk), the [documentation on GitHub](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs), and the links below.

## Who this documentation is for

| Audience | What you will get here |
| --- | --- |
| **Product, compliance, and business readers** | Plain-language model of privacy, where data lives, and what “async” private operations mean in practice. |
| **Architects and technical leads** | End-to-end diagrams, component roles, and boundaries between chains and domains. |
| **Developers** | A clear map from concepts to Solidity/TypeScript work, plus pointers to the authoritative SDK docs. |

## Table of contents

### Understand first (readable without writing code)

1. [What is Privacy on Demand?](what-is-privacy-on-demand.md) — Problem, promise, and constraints in everyday language.
2. [How a private request travels end to end](how-a-private-request-travels-end-to-end.md) — Timeline from user action to decrypted result.
3. [Architecture and main components](architecture-and-components.md) — Where **Inbox**, **MPC executor**, **PodUser**, and **PodLib** sit, with diagrams.
4. [Networks](networks/README.md) — Supported PoD test networks and contract addresses ([COTI Testnet](networks/coti-testnet.md), [Avalanche Fuji](networks/fuji.md)).
5. [Glossary](glossary.md) — Short definitions of terms you will see in PoD and SDK docs.

### Deeper context

6. [Async private operations (why it is not instant)](async-private-operations.md) — What “pending” means and why UX must reflect it.
7. [How do PoA fees work?](how-poa-fees-work.md) — Two-way Inbox budgets, oracle conversion, and step-by-step gas-unit consumption (worked example).
8. [For developers: mapping concepts to the SDK](for-developers-mapping-to-the-sdk.md) — Checklists and links to the [PoD SDK documentation on GitHub](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs).

### Tutorials (hands-on)

9. [Tutorials: building PoD dApps](tutorials-privacy-on-demand.md) — When to use **MpcLib / PodLib** primitives vs **custom COTI + host** contracts, with links to focused walkthroughs.
10. [TypeScript PoD SDK (`CotiPodCrypto`, `PodContract`)](typescript-pod-sdk.md) — Encryption/decryption, fee estimation, method calls, and request ID extraction.
11. [Cookbook: private investor allocations with PoD](cookbook-private-investor-allocations.md) — Start from a familiar public Sepolia allocation dApp, then make allocation reads and withdrawals private with PoD.
12. [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) — Minimal primitive-only adder: `PodUserSepolia`, fees, TypeScript crypto.
13. [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md) — Encrypted messaging shape: `DirectMessageCotiSide` + Sepolia orchestrator.

## Official technical reference

The machine-readable contracts, types, and APIs live in the open-source SDK. Treat this book chapter as the **human-oriented companion**; treat the repository as the **source of truth** for signatures, fees, and network constants:

- [COTI PoD SDK — documentation index](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs)
