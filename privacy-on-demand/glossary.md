# Glossary

Short definitions for **Privacy on Demand** readers. Precise Solidity definitions and type tables are in the [PoD SDK contract types](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/01-it-ct-gt-data-types.md) document.

| Term | Meaning |
| --- | --- |
| **Privacy on Demand (PoD)** | Pattern and tooling for **private computation on COTI** while **orchestrating** from **another EVM chain** via an **Inbox**. |
| **Inbox** | On-chain **message router** on each chain that forwards private jobs toward COTI and delivers callbacks. Implementation in [`coti-pod-inbox-contracts`](https://github.com/coti-io/coti-pod-inbox-contracts). |
| **MPC executor** | COTI-side contract configured as the execution target for **library-style** PoD flows; referenced from your dApp’s routing configuration. |
| **PodUser** | Solidity **configuration mixin** for Inbox address, COTI chain id, and executor address; changes should be **governed** (for example `onlyOwner`). |
| **PodUserSepolia / PodUserFuji** | Network presets that auto-wire inbox and COTI routing in the constructor. |
| **PodLib** | Solidity **helper library** for built-in private operations at 64/128/256-bit widths. |
| **`it*` (input types)** | Encrypted user input with signature material, prepared client-side and sent to your contract. |
| **`gt*` (garbled / compute types)** | Internal private representation during computation on **COTI**—not something your EVM contract should expose as a public API. |
| **`ct*` (ciphertext types)** | Encrypted outputs suitable to store on your chain; users decrypt locally with the account AES key. |
| **PoA / PoD fees** | Native token on your chain funding COTI-side and callback execution budgets for **two-way Inbox** traffic. See [How do PoA fees work?](how-poa-fees-work.md). |
| **Two-way message** | Outbound request to COTI plus inbound callback to your contract; typically needs fee planning for both legs. |
| **Request ID** | 32-byte correlator tying submission to callback; indexed in compact `MessageSent` events. Essential for async UX and troubleshooting. |
| **System error** | Pre-execution Inbox failure (encode / `validateCiphertext`). Delivered on the same `errorSelector(bytes)` as app `raise`. Attributed to `SYSTEM_SENDER`. Detect via `inboxErrorType() == SystemError`. **Not** eligible for `retryFailedRequest`. |
| **`SYSTEM_SENDER`** | Placeholder `originalSender` / `inboxMsgSender()` for system-error return legs. Not a real contract; do not require it to equal your COTI peer. |
| **`inboxErrorType()`** | Inbox view returning `NotErrorContext`, `SystemError`, or `Exception` for the active execution — preferred way for error handlers to branch. |
| **Account AES key** | 32-hex-character user secret for decrypting `ct*` outputs after onboarding; must be handled like credentials. See [Account Onboard](../build-on-coti/guides/account-onboard.md). |
| **PodRequest** | TypeScript helper (`@coti/pod-sdk`) that polls inbox state across chains for async UX. |
| **PodSdkConfig** | JSON config (chains, inbox addresses, RPCs, encryption network) shared by `PodContract` and `PodRequest`. |
| **`@coti-io/coti-contracts`** | npm package with PoD Solidity libraries, interfaces, and examples. |
| **`@coti-io/coti-pod-inbox-contracts`** | npm package with Inbox implementation, fee manager, and miner contracts. |
