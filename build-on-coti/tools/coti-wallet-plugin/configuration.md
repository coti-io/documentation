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
| `snapInstallEnabled` | `boolean` | `true` | When `false`, skip `wallet_requestSnaps` install/connect |
| `defaultNetworkId` | `string` | — | Enforce a specific network chain ID |
| `sepoliaRpcUrl` | `string` | — | Sepolia RPC URL for PoD portal operations |
| `cotiTestnetRpcUrl` | `string` | — | COTI testnet RPC URL for PoD SDK tracking |
| `walletConnectProjectId` | `string` | — | WalletConnect Cloud project ID for RainbowKit |
| `debug` | `boolean` | `false` | Enable verbose internal logging (secrets are never logged) |
| `clearSessionKeyOnWagmiDisconnect` | `boolean` | `false` | Clear in-memory AES key on wagmi disconnect |
| `onboardingServices` | `OnboardingServices` | `{ mode: 'disabled' }` | Grant and encrypted backup service hooks |
| `aesKeyChainId` | `7082400 \| 2632500` | — | COTI chain that owns AES onboarding state |
| `onboardingGrantMinBalanceWei` | `BigNumberish` | `0` | Native COTI threshold before contract onboarding |
| `onboardingGrantPollIntervalMs` | `number` | `2000` | Polling interval after grant callback |
| `onboardingGrantTimeoutMs` | `number` | `60000` | Max wait time after grant callback |
| `additionalSnapAesWriteOrigins` | `string[]` | `[]` | Extra origins allowed to call Snap `set-aes-key` |

{% hint style="info" %}
`aesKeyChainId` accepts only COTI Testnet (`7082400`) or COTI Mainnet (`2632500`). Only COTI chains can hold AES keys.
{% endhint %}

## Onboarding services

Optional host-implemented callbacks for encrypted AES backup storage and native COTI gas grants during contract onboarding.

```tsx
configureCotiPlugin({
  onboardingServices: {
    mode: 'custom',
    fetchEncryptedAesBackup: async ({ address, chainId }) => {
      const res = await fetch(`/aes-backups/${chainId}/${address}`);
      if (res.status === 404) return null;
      return res.json();
    },
    saveEncryptedAesBackup: async ({ address, chainId, backup }) => {
      await fetch(`/aes-backups/${chainId}/${address}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(backup),
      });
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

### `OnboardingServices` modes

| Mode | Behavior |
| --- | --- |
| `disabled` | No grant or backup features (default) |
| `custom` | Use the provided callback functions |
| `official` | Reserved for stable COTI-hosted APIs |

### Callback reference

| Callback | Purpose |
| --- | --- |
| `fetchEncryptedAesBackup` | `GET /aes-backups/:chainId/:address` — returns `EncryptedAesBackup` or `null` |
| `saveEncryptedAesBackup` | `PUT /aes-backups/:chainId/:address` — stores encrypted backup |
| `replaceEncryptedAesBackup` | Replace an existing encrypted backup |
| `grantNativeCoti` | `POST` with `{ address, chainId }` — funds wallet for onboarding gas |

### `EncryptedAesBackup` shape

```typescript
interface EncryptedAesBackup {
  version: 1;
  address: string;
  chainId: number;
  signatureKind: 'eip712';
  iv: string;
  ciphertext: string;
  createdAt: string;
}
```

Backup restore requires a wallet EIP-712 signature — a stored blob alone is not enough to recover the AES key.

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
  snapInstallEnabled: true,
});
```

### Local Snap development

When developing against a local `coti-snap` server:

```tsx
configureCotiPlugin({
  snapId: 'local:http://localhost:8080',
  snapInstallEnabled: true,
});
```

### Disable Snap install prompts

If the Snap is pre-installed and you want to skip install/connect prompts:

```tsx
configureCotiPlugin({
  snapInstallEnabled: false,
});
```

An already-installed Snap can still be used for AES key retrieval.

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
| `VITE_COTI_SNAP_INSTALL_ENABLED` | Enable/disable Snap install |
| `VITE_AES_BACKUP_API_URL` | Remote encrypted backup API base URL |
| `VITE_GRANT_API_URL_TESTNET` | Testnet native COTI grant API |
| `VITE_GRANT_API_URL_MAINNET` | Mainnet native COTI grant API |
| `VITE_ONBOARDING_GRANT_MIN_BALANCE_COTI` | Grant threshold (default `0.2`) |

Run the example with local Snap:

```bash
npm run dev:local-snap
```

This starts the local `coti-snap` server, Snap companion dApp, and wallet example with `VITE_SNAP_ID=local:http://localhost:8080`.

## Security notes

* The active AES key lives in session-only React state, wallet-bound to prevent cross-account leakage.
* Encrypted backups are optional and host-defined.
* `debug: true` enables verbose logging but **never** logs secret material (AES keys, ciphertext, signatures).
* Set `clearSessionKeyOnWagmiDisconnect: true` for stricter shared-browser security at the cost of re-fetching the key on reconnect.

## Related docs

* [Integration Guide](integration-guide.md)
* [API Reference](api-reference.md)
