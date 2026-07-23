# Troubleshooting

### Common Errors and Fixes

| Error                             | Cause                                         | Fix                                                   |
| --------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| `PublicAmountsDisabled`           | `publicAmountsEnabled` is false               | Use encrypted variants or re-enable via admin         |
| `ERC20SelfTransferNotAllowed`     | `from == to` in transfer                      | Never transfer to yourself                            |
| `ERC20InvalidMetadata`            | Empty name or symbol in constructor           | Pass non-empty strings                                |
| `ERC20: mint failed`              | MPC precompile returned false                 | Check MPC network status; retry                       |
| `AES key mismatch`                | Wrong AES key used to decrypt                 | Re-onboard wallet to get correct key                  |
| `invalid BytesLike value`         | Passing HardhatEthersSigner to `prepareIT256` | Use `new ethers.Wallet(privateKey, provider)` instead |
| `PRIVATE_AES_KEY_TESTNET not set` | Missing env var                               | Add 32 hex char key to `.env` (no `0x` prefix)        |
| Deposit stuck `Pending` forever   | Several distinct causes (see below)           | Diagnose with pToken `requests` / `failedRequests` and Inbox `errors` / `getOutboxError` |
| Withdraw stuck `TransferPending`  | Transfer-to-portal request Failed                 | Call `PrivacyPortal.cancelFailedWithdrawal(withdrawalId)`; do not expect underlying release |
| Encode/`validateCiphertext` fail (`failedRequests` as `{ErrorData}` with `errorCode == 2`) | Wrong `it*` signer / encode fail | Clear UI pending; submit a **new** op (not COTI `retryFailedRequest`) |
| Mother registration / `FactoryNotAllowed` | Factory not allowlisted on COTI mother before registration mined | Allowlist factory (`setAllowedFactory`), then permissionless COTI `retryFailedRequest` for the failed registration id |
| Deposits before mother confirms | Portal created but mother `isRegistered` still false | Keep deposits disabled / pause until mother registration confirms |

### Stuck `Pending` deposits — diagnose before refunding

A deposit mint can remain **`Pending`** for different reasons. Do **not** assume every stuck mint is a system-failed encode:

| Observation | Likely cause | Action |
| --- | --- | --- |
| `requests(id).status == SystemFailed` / `failedRequests` decodes to Inbox `{ErrorData}` code `2` | Encode / `validateCiphertext` failed before COTI mint logic | Clear UI pending; call `refundFailedDeposit` (permissionless) after SystemFailed |
| COTI Inbox `errors[id].errorCode == 1` | Target execution reverted (retryable) | Call permissionless `retryFailedRequest` on COTI; do **not** refund while mint may still succeed |
| No COTI incoming request / never mined | Relayer lag or miner not ingesting | Wait for miner; do not treat as SystemFailed |
| Mother never registered / factory not allowlisted | Registration one-way failed or never confirmed | Fix allowlist + `retryFailedRequest`; keep user deposits off until `isRegistered` |

**Break-glass:** some deployments expose an admin refund for still-`Pending` escrows. That is operationally dangerous if COTI mint can still succeed afterward—prefer SystemFailed refunds and retries.

### Privacy Portal recovery

* **System-failed deposit (mint):** underlying stays escrowed until anyone calls `refundFailedDeposit(mintRequestId)` after `pToken.requests(id).status == SystemFailed` (funds always return to the depositor). App `raise` / `Failed` is **not** refundable (mint should not raise). Portal protocol fee is kept.
* **Failed withdraw (transfer):** after the transfer request is `Failed` or `SystemFailed`, call `cancelFailedWithdrawal(withdrawalId)` to mark the withdrawal `Failed`. Underlying is **not** released; the user still holds pTokens; portal fee is kept.
* **Factory → mother registration:** `createPortal` **submits** a one-way registration message; it does **not** wait for the mother to confirm. Keep deposits disabled until `PodErc20CotiMother.isRegistered(sourceChainId, pToken)` is true. If registration failed with `FactoryNotAllowed` before allowlist, allowlist the factory then call COTI `retryFailedRequest`.

### Security Notes

* Deploy only on chains where the MPC precompile at `address(0x64)` is trusted.
* `MINTER_ROLE` must only be granted to audited contracts&#x20;
* Encrypted operations (`mintGt`, `burnGt`, `transferGT`) return `gtBool` and do NOT revert on failure — always decrypt and check the return value.
* Self-transfers (`from == to`) are explicitly blocked at the contract level.
* `transferAndCall` is protected by `nonReentrant` but the callback contract must be trusted.
* `totalSupply()` always returns `0` for privacy — do not rely on it for supply accounting.
* Inbox `executed` / compact response events mean the return leg was **received**, not that your app callback committed—confirm via pToken / portal status.
