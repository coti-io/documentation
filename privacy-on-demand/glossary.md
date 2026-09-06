# Glossary

Short definitions for **Privacy on Demand** readers. Precise Solidity definitions and type tables are in the [PoD SDK contract types](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/01-it-ct-gt-data-types.md) document.

| Term | Meaning |
| --- | --- |
| **Privacy on Demand (PoD)** | Pattern and tooling for **private computation on COTI** while **orchestrating** from **another EVM chain** via an **Inbox**. |
| **Inbox** | **On-chain router** on your EVM chain that **forwards** private jobs toward COTI and **delivers callbacks** with results to your contract. |
| **MPC executor** | **COTI-side contract** configured as the execution target for **library-style** PoD flows; referenced from your dApp’s routing configuration. |
| **PodUser** | Solidity **configuration mixin** for **Inbox address**, **COTI chain id**, and **executor address**; changes should be **governed** (for example `onlyOwner`). |
| **PodLib** | Solidity **helper library** for **common private operations** (fixed-width arithmetic and comparisons in supported paths). |
| **`it*` (input types)** | **Encrypted user input** with required **signature material**, prepared client-side and sent to your contract. |
| **`gt*` (garbled / compute types)** | **Internal private representation** during computation on **COTI**—not something your EVM contract should expose as a public API. |
| **`ct*` (ciphertext types)** | **Encrypted outputs** suitable to **store on your chain**; users **decrypt locally** with **account AES** keys where applicable. |
| **PoA fees** | In this book, **Privacy on Demand (PoD) fees** for **two-way Inbox** traffic: native token on your chain that funds **COTI-side** and **callback** execution budgets. See [How do PoA fees work?](how-poa-fees-work.md). |
| **Two-way message** | Inbox flow: **outbound** request to COTI plus **inbound callback** to your contract; typically needs **fee** planning for both legs. |
| **Request ID** | Correlator tying a **submission** to a **callback**; essential for **async** UX and troubleshooting. |
| **Account AES key** | User-side secret material used to **decrypt** many `ct*` outputs after onboarding; must be **handled like credentials**. |
