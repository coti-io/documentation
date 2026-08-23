# Audit Reports

Security is a top priority for the COTI network. Below you’ll find a list of independent security audits conducted on different components of the COTI ecosystem. Each report outlines the scope, focus areas, and includes a link to the full audit documentation.

These audits play a critical role in ensuring that the COTI protocol remains secure, reliable, and ready for real-world adoption.

You can find all published audit reports here:

| Date      | Reviewer | Scope           | Link                                                                                                                  |
| --------- | -------- | --------------- | --------------------------------------------------------------------------------------------------------------------- |
| 2025 - 4  | Sayfer   | Account Onboard | [2025\_Sayfer](Sayfer_gCoti_AccountOnBoard.pdf)                                                                       |
| 2025 - 4  | Sayfer   | gCOTI           | [2025\_Sayfer](Sayfer_gCoti_AccountOnBoard.pdf)                                                                       |
| 2025 - 3  | Hacken   | MPC             | [2025\_Hacken](Hacken_MPC.pdf)                                                                                        |
| 2025 - 8  | Hacken   | GoEthereum      | [2025\_Hacken](Hacken_Go_Ethereum.pdf)                                                                                |
| 2025 - 11 | Sayfer   | Redeemer        | [2025\_Sayfer](Sayfer_Redeemer.pdf)                                                                                   |
| 2026 - 3  | Sayfer   | Private ERC20   | [2026\_ERC20\_Sayfer](Sayfer-2026-03-Smart-Contract-Audit-Report-for-Coti.pdf)                                        |
| 2026 - 3  | Sayfer   | Metamask Snap   | [2026\_Metamask](https://github.com/coti-io/coti-snap/blob/main/docs/Sayfer-2026-03-Metamask-Snap-Audit-for-Coti.pdf) |
| 2026 - 5  | Sayfer   | Privacy Portal  | [2026\_PrivacyPortal](Sayfer-2026-05-Smart-Contract-Audit-Report-for-Coti.pdf)                                        |

## PoD Inbox and Privacy Portal hardening (internal review)

In addition to the external reports above, the PoD **Inbox** (`coti-pod-inbox-contracts`) and **PoD Privacy Portal** stack received an internal security review with follow-up fixes. Integrators should treat the following as current behavior (also reflected in the PoD / Avalanche books):

| Area | What changed for integrators |
| --- | --- |
| **Capped returndata** | Execution failures store at most 256 bytes of returndata. **`getOutboxError`** returns `(code, data)` with those raw bytes for client-side decoding. |
| **`retryFailedRequest`** | Permissionless while error code is `1`. Encode failure on retry **reverts** and preserves code `1` (does not flip to encode-failed). |
| **One-way `errorSelector`** | `sendOneWayMessage` rejects non-zero `errorSelector`—use two-way for error callbacks. |
| **`executed` / response events** | Mean the return leg was **ingested**, not that the app callback committed. |
| **Fee gas price** | On-chain budgets use **bounded reference** gas price (`setGasPriceBounds`), not unbounded tip manipulation. |
| **Oracle cache** | `refreshCache()` refreshes **both** inbox legs; configure with `setInboxTokens` (Uniswap oracles set legs from pairs at construction). |
| **Portal deposits** | Prefer `refundFailedDeposit` only after **SystemFailed**. Stuck `Pending` may be retryable execution failure or miner lag—diagnose before refunding. Keep deposits off until mother registration confirms. |

See [Async private operations](../privacy-on-demand/async-private-operations.md), [How do PoA fees work?](../privacy-on-demand/how-poa-fees-work.md), and [Privacy Portal troubleshooting](../coti-privacy-portal/developer-guide/troubleshooting.md).
