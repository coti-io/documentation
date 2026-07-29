# AES Key Onboarding

Onboarding retrieves or restores the wallet-bound AES key used to decrypt private token balances. The active key is kept in React session state, while optional restore/save services can store an encrypted backup outside the session.

Application UI should use the provider-level private unlock controller:

```tsx
<PrivacyBridgeProvider privateUnlock={{ theme, warning }}>
  <App />
</PrivacyBridgeProvider>
```

Then call `usePrivateUnlock().unlock()`, `usePrivateUnlock().lock()`, or `usePrivateUnlock().requireUnlock(action)` from app components. Do not orchestrate unlock directly with `refreshPrivateBalances({ restoreOnly: true })` or a locally owned `OnboardModal`.

## Onboarding routes

| Wallet | First route | Fallback |
| --- | --- | --- |
| MetaMask with Snap | Retrieve AES key from Snap | Contract onboarding if Snap is empty or unavailable |
| Other wallets / MetaMask without usable Snap | Encrypted backup restore, then contract onboarding | Manual AES key input in `OnboardModal` if the host supplies `onManualAesKeySubmit` |

## Contract onboarding flow

1. Caller invokes the AES key provider with the connected wallet address.
2. If `onboardingServices.fetchEncryptedAesBackup` is configured, the plugin emits `restoring-backup`, fetches the encrypted backup, asks the wallet for the EIP-712 restore signature, and decrypts it.
3. If `restoreOnly` is true and no backup is restored, the flow returns without contract onboarding.
4. The plugin switches to COTI mainnet or testnet when needed, muting chain-change reactions during the temporary switch.
5. If `grantNativeCoti` is configured and the wallet balance is below the minimum threshold, the plugin calls the grant service and polls native COTI balance.
6. The plugin creates a `@coti-io/coti-ethers` `BrowserProvider` and signer.
7. `signer.generateOrRecoverAes()` runs the onboarding flow. The wallet sees the message signature and, for first-time onboarding, the on-chain transaction.
8. The plugin reads and validates the AES key, then switches the wallet back to the original chain when applicable.
9. For MetaMask, the plugin attempts to persist the key into the Snap if the origin is allowed.
10. If **Save Locally** is enabled and backup save callbacks are configured, the plugin encrypts the AES key and calls `saveEncryptedAesBackup` or `replaceEncryptedAesBackup`. When Snap storage is used for onboarding, encrypted backup save is skipped (the Snap already holds the key).
11. The flow completes and returns the AES key. Unlock is only treated as successful after private balances refresh with the session key.

## Save Locally and progress UI

Non-Snap wallets show a **Save Locally** switch card in `OnboardModal` (encrypted backup blob; restore still needs a wallet signature). MetaMask Snap onboarding hides that switch because the Snap persists the key.

| Situation | Persist step in progress UI | Encrypted backup save |
| --- | --- | --- |
| Non-Snap, **Save Locally** on | Shown (`persisting-key` / `saving-backup`) | Runs when backup services are configured |
| Non-Snap, **Save Locally** off | Hidden | Skipped |
| Snap onboarding | Shown (Snap always persists the key) | Skipped; key goes to Snap |

## Example app services

The [example app](example-app.md) configures `onboardingServices` with `mode: 'custom'` and implements the backup callbacks with browser `localStorage` (`coti-example:aes-backup:<chainId>:<address>`). That is the **supported** encrypted-backup path.

| Env var | Behavior |
| --- | --- |
| `VITE_GRANT_API_URL_TESTNET` | Overrides testnet grant API URL (plugin has a default testnet grant endpoint) |
| `VITE_GRANT_API_URL_MAINNET` | Mainnet grant API URL (no default — grant skipped on mainnet until set) |
| `VITE_ONBOARDING_GRANT_ENABLED=false` | Skips grant requests; wallet must already have native COTI for gas |

The example keeps the active AES key in memory. LocalStorage only holds the encrypted backup blob, not the live session key.

Manual AES key input uses the same encrypted backup helper as contract onboarding when **Save Locally** is on: the user signs the backup context, the key is encrypted, and the encrypted blob is saved through the configured host callbacks. If the user rejects that backup signature, the plugin cancels local save, skips the AES key success screen, and still completes unlock with the session key when balance refresh succeeds.

## Supported wallets for encrypted backup

Signature-derived encrypted backup is a **compatibility fallback**, not a universal recovery mechanism. The plugin does not persist backups itself — the host supplies `onboardingServices` callbacks (`mode: 'custom'`). The **only supported** store is browser **`localStorage`**. Remote AES backup is **deprecated**.

Restore works only when the host can fetch the blob from localStorage and the wallet later reproduces **identical effective signing material** for the same EIP-712 backup message. Controlling the same address later is **not** enough by itself. localStorage backups are same-browser / same-origin only.

Preferred storage order:

1. Wallet-native protected storage (MetaMask Snap).
2. A dedicated deterministic encryption keypair separated from the transaction-signing key (when the wallet exposes one).
3. Signature-derived encrypted backup for verified wallets only.

### Officially supported

EOA wallets that:

* implement **deterministic ECDSA** for EIP-712 (`eth_signTypedData_v4`);
* keep stable signature serialization for the same digest;
* use a fixed signing key for the account (no rotation during the backup lifecycle).

In practice this includes common software EOAs such as MetaMask (extension) when signing with a standard imported or generated account key.

### Not officially supported

Do **not** rely on signature-derived backup for:

* randomized ECDSA implementations;
* MPC wallets;
* multisig wallets;
* ERC-1271 smart accounts;
* passkey wallets;
* rotating-key accounts;
* hardware wallets whose firmware may change signature behavior;
* wallets that change signature serialization across versions.

Before saving a backup, the plugin requests a second independent signature and confirms the new blob decrypts. If the wallet cannot reproduce the signature, the backup is **not** persisted and the plugin returns `AES_BACKUP_WALLET_NOT_SUPPORTED`. Prefer Snap (or equivalent) for unsupported wallets when available.

Avoid wording such as “recoverable from any wallet using the same address.”

## Security properties

1. The active AES key is session-only React state and is wallet-bound to prevent cross-account leakage.
2. Encrypted backups are optional and host-defined through `configureCotiPlugin`. The user can turn **Save Locally** off for non-Snap flows.
3. A stored blob alone is not enough to recover the AES key, but possession of **both** the encrypted backup and a matching EIP-712 wrap signature is sufficient. Treat that signature as a sensitive unlock action.
4. **Cross-app crypto portability.** Origin binding is omitted so the same encrypted blob could be unlocked in another trusted COTI plugin app if that app had the blob. With localStorage-only persistence, each origin keeps its own store — remote shared backup is deprecated. Domain separation is not origin security; only sign the unlock request in official or explicitly trusted COTI applications. See [AES backup security model](aes-backup-security.md#cross-app-restore-crypto-portability).
5. Manual AES key input is session-only unless **Save Locally** (or Snap persistence) stores it.
6. Locking private balances clears plaintext session AES state and hides balances. Wagmi disconnect does the same by default (`clearSessionKeyOnWagmiDisconnect: true`). Snap keys and encrypted backups remain intact for the next unlock.

## Error handling

* User rejection during Snap, restore signature, chain switch, or onboarding returns without storing a key.
* Backup restore failures fall through to contract onboarding and set a non-blocking warning.
* Backup save failures do not block a successful onboarding; the user receives a warning.
* If the wallet fails the backup determinism check, the plugin does not save the blob and surfaces `AES_BACKUP_WALLET_NOT_SUPPORTED`.
* Rejecting the manual **Save Locally** backup signature cancels local save and skips the success screen; unlock can still finish if private balances refresh.
* Unlock does not succeed if private balances fail to refresh after the AES key is available.
* Grant API HTTP errors are treated as skipped grants. The onboarding transaction still needs native COTI gas, so an unfunded wallet fails through the normal insufficient-balance path.
* Invalid AES key format sets an onboarding error state.

## Related docs

* [Integration Guide](integration-guide.md)
* [Configuration](configuration.md)
* [AES backup security model](aes-backup-security.md)
* [Example App](example-app.md)
