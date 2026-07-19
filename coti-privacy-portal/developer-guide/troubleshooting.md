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
| Deposit stuck `Pending` forever   | Mint encode/`validateCiphertext` failed without UI handling | Poll pToken `requests` / `failedRequests`; call `PrivacyPortal.refundFailedDeposit(requestId)` after **SystemFailed** |
| Withdraw stuck `TransferPending`  | Transfer-to-portal request Failed                 | Call `PrivacyPortal.cancelFailedWithdrawal(withdrawalId)`; do not expect underlying release |
| Encode/`validateCiphertext` fail (`failedRequests` as `{ErrorData}` with `errorCode == 2`) | Wrong `it*` signer / encode fail | Clear UI pending; submit a **new** op (not COTI `retryFailedRequest`) |

### Privacy Portal recovery

* **System-failed deposit (mint):** underlying stays escrowed until anyone calls `refundFailedDeposit(mintRequestId)` after `pToken.requests(id).status == SystemFailed` (funds always return to the depositor). App `raise` / `Failed` is **not** refundable (mint should not raise). Portal protocol fee is kept.
* **Failed withdraw (transfer):** after the transfer request is `Failed` or `SystemFailed`, call `cancelFailedWithdrawal(withdrawalId)` to mark the withdrawal `Failed`. Underlying is **not** released; the user still holds pTokens; portal fee is kept.

### Security Notes

* Deploy only on chains where the MPC precompile at `address(0x64)` is trusted.
* `MINTER_ROLE` must only be granted to audited contracts&#x20;
* Encrypted operations (`mintGt`, `burnGt`, `transferGT`) return `gtBool` and do NOT revert on failure — always decrypt and check the return value.
* Self-transfers (`from == to`) are explicitly blocked at the contract level.
* `transferAndCall` is protected by `nonReentrant` but the callback contract must be trusted.
* `totalSupply()` always returns `0` for privacy — do not rely on it for supply accounting.
