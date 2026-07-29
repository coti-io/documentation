# Configuration

Configure the COTI Wallet Plugin at initialization time via `configureCotiPlugin()`. Call this **before** rendering any plugin hooks.

```tsx
import { configureCotiPlugin } from '@coti-io/coti-wallet-plugin';

configureCotiPlugin({
  snapId: 'npm:@coti-io/coti-snap',
  aesKeyChainId: 7082400,
  walletConnectProjectId: 'your-walletconnect-project-id',
  debug: false,
});
```

## `CotiPluginConfig` options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `snapId` | `string` | `'npm:@coti-io/coti-snap'` | COTI MetaMask Snap ID |
| `snapVersion` | `string` | — | Optional Snap version for `wallet_requestSnaps` |
| `snapEnabled` | `boolean` | `true` | When `false`, Snap is fully disabled (no install/probe/key use) |
| `defaultNetworkId` | `string` | — | Enforce a specific network chain ID |
| `sepoliaRpcUrl` | `string` | — | Sepolia RPC URL for PoD portal operations |
| `cotiTestnetRpcUrl` | `string` | — | COTI testnet RPC URL for PoD SDK tracking |
| `walletConnectProjectId` | `string` | — | WalletConnect Cloud project ID for RainbowKit |
| `debug` | `boolean` | `false` | Enable verbose internal logging (secrets are never logged) |
| `clearSessionKeyOnWagmiDisconnect` | `boolean` | `true` | Clear in-memory AES key (and Snap cache) on wagmi disconnect |
| `onboardingServices` | `OnboardingServices` | `{ mode: 'disabled' }` | Grant and encrypted backup service hooks |
| `aesKeyChainId` | `7082400 \| 2632500` | — | COTI chain that owns AES onboarding state |
| `onboardingGrantEnabled` | `boolean` | `true` | When `false`, skip native COTI grant requests |
| `onboardingGrantMinBalanceWei` | `BigNumberish` | `0.2 COTI` | Native COTI threshold before contract onboarding |
| `onboardingGrantPollIntervalMs` | `number` | `2000` | Polling interval after grant callback |
| `onboardingGrantTimeoutMs` | `number` | `60000` | Max wait time after grant callback |
| `additionalSnapAesWriteOrigins` | `string[]` | `[]` | Extra origins allowed to call Snap `set-aes-key` |
| `unsafeSkipBackupDeterminismCheck` | `boolean` | `false` | **Unsafe** — skip second-signature restore test before persist |

{% hint style="info" %}
`aesKeyChainId` accepts only COTI Testnet (`7082400`) or COTI Mainnet (`2632500`). Only COTI chains can hold AES keys.
{% endhint %}

## Onboarding services

Optional host-implemented callbacks for encrypted AES backup storage and native COTI gas grants during contract onboarding.

The plugin encrypts and decrypts AES backup blobs. It does **not** persist them itself. The **only supported** backup store is browser **`localStorage`**, wired through `mode: 'custom'` callbacks. **Remote AES backup is deprecated.** See [AES backup security model](aes-backup-security.md).

```tsx
const backupKey = (address: string, chainId: number) =>
  `my-app:aes-backup:${chainId}:${address.toLowerCase()}`;

configureCotiPlugin({
  onboardingServices: {
    mode: 'custom',
    fetchEncryptedAesBackup: async ({ address, chainId }) => {
      const raw = localStorage.getItem(backupKey(address, chainId));
      return raw ? JSON.parse(raw) : null;
    },
    saveEncryptedAesBackup: async ({ address, chainId, backup }) => {
      localStorage.setItem(backupKey(address, chainId), JSON.stringify(backup));
    },
    replaceEncryptedAesBackup: async ({ address, chainId, backup }) => {
      localStorage.setItem(backupKey(address, chainId), JSON.stringify(backup));
    },
    deleteEncryptedAesBackup: async ({ address, chainId }) => {
      localStorage.removeItem(backupKey(address, chainId));
    },
    grantNativeCoti: async ({ address, chainId }) => {
      const res = await fetch('https://your-grant-api.example.com', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ address, chainId }),
      });
      return res.json();
    },
  },
});
```

When `grantNativeCoti` is omitted, the plugin may still use the built-in grant URL config (`grantApiUrlTestnet` / `grantApiUrlMainnet`) if grants are enabled.

### `OnboardingServices` modes

| Mode | Behavior |
| --- | --- |
| `disabled` | No grant/backup features (default) |
| `custom` | Use the provided callback functions (backup callbacks should use `localStorage`) |
| `official` | Reserved for stable COTI-hosted APIs |

There is no built-in `mode: 'localStorage'`. The [example app](example-app.md) shows the supported `localStorage` callback pattern.

### Callback reference

| Callback | Purpose |
| --- | --- |
| `fetchEncryptedAesBackup` | Return `EncryptedAesBackup` or `null` for `{ address, chainId }` |
| `saveEncryptedAesBackup` | Persist a new encrypted backup |
| `replaceEncryptedAesBackup` | Replace an existing encrypted backup |
| `deleteEncryptedAesBackup` | Remove a stored backup (best-effort; e.g. after rejecting an outdated blob) |
| `grantNativeCoti` | Fund wallet for onboarding gas (`{ address, chainId }`) → `GrantResult` |

### `EncryptedAesBackup` shape

```typescript
interface EncryptedAesBackup {
  version: 2;
  address: string;
  chainId: number;
  signatureKind: 'eip712';
  kdf: 'hkdf-sha256';
  iv: string;
  ciphertext: string;
  createdAt: string;
}
```

Backup restore requires a wallet EIP-712 signature — a stored blob alone is not enough to recover the AES key. Signature-derived backup is only reliable for wallets that reproduce identical EIP-712 signing material; see [Supported wallets for encrypted backup](aes-key-onboarding.md#supported-wallets-for-encrypted-backup) and the full [AES backup security model](aes-backup-security.md).

### `GrantResult` shape

```typescript
interface GrantResult {
  txHash?: string;
  amountWei?: string;
  status?: 'submitted' | 'funded' | 'skipped';
}
```

## Snap configuration

### Production Snap

```tsx
configureCotiPlugin({
  snapId: 'npm:@coti-io/coti-snap',
  snapEnabled: true,
});
```

### Local Snap development

When developing against a local `coti-snap` server:

```tsx
configureCotiPlugin({
  snapId: 'local:http://localhost:8080',
  snapEnabled: true,
});
```

### Disable Snap entirely

```tsx
configureCotiPlugin({
  snapEnabled: false,
});
```

Unlock continues via encrypted backup restore and/or contract onboarding.

### Additional Snap write origins

Whitelist dApp domains that call Snap `set-aes-key` outside the published COTI portals:

```tsx
configureCotiPlugin({
  additionalSnapAesWriteOrigins: ['https://portal.example.com'],
});
```

The Snap manifest's `allowedOrigins` must also include these domains.

## Example app environment variables

The [example app](https://github.com/coti-io/coti-wallet-plugin/tree/main/examples) uses Vite env vars for local development. Production dApps should use `configureCotiPlugin()` instead.

| Variable | Purpose |
| --- | --- |
| `VITE_WALLETCONNECT_PROJECT_ID` | WalletConnect Cloud project ID |
| `VITE_SNAP_ID` | Local Snap dev (`local:http://localhost:8080`) |
| `VITE_COTI_SNAP_ENABLED` | Enable/disable Snap (`false` disables install, probe, and key use) |
| `VITE_ONBOARDING_GRANT_ENABLED` | Enable/disable native COTI grant requests (default `true`) |
| `VITE_GRANT_API_URL_TESTNET` | Testnet native COTI grant API |
| `VITE_GRANT_API_URL_MAINNET` | Mainnet native COTI grant API |
| `VITE_ONBOARDING_GRANT_MIN_BALANCE_COTI` | Grant threshold (default `0.2`) |

Encrypted AES backups in the example use host `localStorage` via `onboardingServices` callbacks (`coti-example:aes-backup:<chainId>:<address>`). Remote AES backup is deprecated.

Run the example with local Snap:

```bash
npm run dev:local-snap
```

This starts the local `coti-snap` server, Snap companion dApp, and wallet example with `VITE_SNAP_ID=local:http://localhost:8080`.

## Security notes

* The active AES key lives in session-only React state, wallet-bound to prevent cross-account leakage.
* Encrypted backups are optional; supported persistence is host `localStorage` via `onboardingServices` (remote backup is deprecated).
* `debug: true` enables verbose logging but **never** logs secret material (AES keys, ciphertext, signatures).
* Set `clearSessionKeyOnWagmiDisconnect: false` only if you want reconnect of the same wallet to keep the in-memory key (weaker on shared browsers).

## Related docs

* [Integration Guide](integration-guide.md)
* [API Reference](api-reference.md)
