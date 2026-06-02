# Wallet Integration

The COTI Wallet Plugin supports multiple wallet types through wagmi v2's connector system. Wallet detection determines how AES keys are retrieved — via the COTI Snap for MetaMask or via the Onboarding Contract for all other wallets.

## Supported Wallet Types

| WalletType         | Connector IDs Matched                          | AES Key Strategy         |
| ------------------ | ---------------------------------------------- | ------------------------ |
| `'metamask'`       | `metaMask`, `io.metamask`, `io.metamask.flask` | COTI Snap (if installed) |
| `'coinbase'`       | `coinbaseWalletSDK`, `*coinbase*`              | Onboarding Contract      |
| `'walletconnect'`  | `walletConnect`, `*walletconnect*`             | Onboarding Contract      |
| `'rainbow'`        | `rainbow`, `*rainbow*`                         | Onboarding Contract      |
| `'phantom'`        | `phantom`, `*phantom*`                         | Onboarding Contract      |
| `'trust'`          | `trust-extension`, `trustWallet`, `*trust*`    | Onboarding Contract      |
| `'rabby'`          | `rabby`, `*rabby*`                             | Onboarding Contract      |
| `'ledger'`         | `ledger`, `*ledger*`                           | Onboarding Contract      |
| `'unknown'`        | Any unrecognized connector ID                  | Onboarding Contract      |

{% hint style="info" %}
Patterns with `*` indicate case-insensitive partial matching as a fallback (e.g. `*metamask*` matches `io.metamask.flask`).
{% endhint %}

## How Wallet Detection Works

1. When a user connects via RainbowKit, wagmi exposes a stable `connector.id` (e.g. `"io.metamask.flask"`, `"coinbaseWalletSDK"`).
2. `mapConnectorIdToWalletType(connectorId)` first attempts an **exact match** against the `CONNECTOR_ID_TO_WALLET_TYPE` map.
3. If no exact match is found, it performs **case-insensitive partial matching** (e.g. any ID containing `"metamask"` resolves to `'metamask'`).
4. If the resolved type is `'metamask'`, an async check via `wallet_getSnaps` determines whether the COTI Snap is installed, setting `isMetaMaskWithSnap`.
5. The `useAesKeyProvider` hook uses `isMetaMaskWithSnap` to route: Snap flow for MetaMask, Onboarding Contract for everything else.

## Adding a New Wallet

To add support for a newly recognized `connector.id` or new wallet definition:

### 1. Add to Type

Add the new wallet identifier to the `WalletType` union in `src/hooks/useWalletType.ts`:

```typescript
export type WalletType = 'metamask' | 'coinbase' | ... | 'okx' | 'unknown';
```

### 2. Map Connector ID

Add wagmi's `connector.id` for that wallet to the `CONNECTOR_ID_TO_WALLET_TYPE` constant:

```typescript
const CONNECTOR_ID_TO_WALLET_TYPE: Record<string, WalletType> = {
  ...
  'com.okex.wallet': 'okx',
  okx: 'okx',
};
```

### 3. Add Partial Match (optional)

If the wallet has multiple connector ID variants, add a partial match case in `mapConnectorIdToWalletType`.

### 4. Verify Fallback

If a connector ID isn't mapped, it automatically falls back to `'unknown'`. The `'unknown'` type uses the standard EIP-1193 Onboarding Contract fallback method by default, meaning most wallets will "just work" even if not explicitly mapped.

{% hint style="info" %}
Adding a new wallet type is only necessary if you need to implement wallet-specific behavior. For standard EIP-1193 wallets that use the Onboarding Contract flow, no changes are needed — they work automatically via the `'unknown'` fallback.
{% endhint %}
