# Glossary

Short definitions for **Privacy on Demand** readers. Precise Solidity definitions and type tables are in the [PoD SDK contract types](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/01-it-ct-gt-data-types.md) document.

| Term | Meaning |
| --- | --- |
| **Privacy on Demand (PoD)** | Pattern and tooling for **private computation on COTI** while **orchestrating** from **another EVM chain** via an **Inbox**. |
| **Privacy on Avalanche** | PoD applied with **Avalanche Fuji C-Chain** as the host (AVAX fees, SnowScan, `PodUserFuji`). |
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
| **System error** | Pre-execution Inbox failure (encode / `validateCiphertext`). Delivered on the same `errorSelector(bytes)` as app `raise`. Attributed to `SYSTEM_SENDER`. Detect via `inboxErrorType() == SystemError`. **Not** eligible for `retryFailedRequest`. |
| **`SYSTEM_SENDER`** | Placeholder `originalSender` / `inboxMsgSender()` for system-error return legs. Not a real contract; do not require it to equal your COTI peer. |
| **`inboxErrorType()`** | Inbox view returning `NotErrorContext`, `SystemError`, or `Exception` for the active execution — preferred way for error handlers to branch. |
| **Execution failure (code `1`)** | Target ran and reverted without `raise`. Stored on COTI with capped returndata; **permissionless** `retryFailedRequest` while code remains `1`. |
| **`getOutboxError`** | View that returns `(code, data)` — for execution failures, the **raw capped returndata** (≤256 bytes). Decode in the client. |
| **`executed` / `IncomingResponseReceived`** | Mean the **return or error leg was ingested**—not that the application callback committed. |
| **One-way message** | Outbound-only Inbox send. **Cannot** register a non-zero `errorSelector` (use two-way if you need error callbacks). |
| **`retryFailedRequest`** | Permissionless COTI Inbox call that re-executes a request still marked execution-failed (code `1`). Encode failure on retry **reverts** and preserves code `1`. |
| **Gas-price bounds** | Operator-configured floor / ceiling / min priority used when converting fee AVAX into gas-unit budgets (bounded reference price—not raw tip manipulation). |
| **Account AES key** | User-side secret material used to **decrypt** many `ct*` outputs after onboarding; must be **handled like credentials**. |
