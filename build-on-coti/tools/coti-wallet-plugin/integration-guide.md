# Integration Guide

This guide walks through integrating the COTI Wallet Plugin into a React/wagmi dApp.

**Principle:** The provider owns the unlock flow. App pages call commands. Do not build custom unlock routing.

## 1. Wrap your app once

Mount `WagmiRainbowKitProvider` and `PrivacyBridgeProvider` near the root of your app:

```tsx
import {
  PrivacyBridgeProvider,
  WagmiRainbowKitProvider,
  type OnboardModalTheme,
} from '@coti-io/coti-wallet-plugin';

const onboardTheme: OnboardModalTheme = {
  modal: { backgroundColor: '#ffffff', color: '#0f172a' },
  title: { color: '#0f172a' },
  description: { color: '#64748b' },
  primaryButton: { backgroundColor: '#1E29F6', color: '#ffffff' },
  checkboxText: { color: '#0f172a' },
  tooltipButton: { color: '#64748b' },
};

export function Root() {
  return (
    <WagmiRainbowKitProvider walletConnectProjectId={walletConnectProjectId}>
      <PrivacyBridgeProvider
        privateUnlock={{
          theme: onboardTheme,
          warning:
            'This dApp never stores or receives the AES key. Unlock stays inside the plugin.',
          onRestoreCancelled: () => {
            // Optional: show "User canceled" toast.
          },
        }}
      >
        <App />
      </PrivacyBridgeProvider>
    </WagmiRainbowKitProvider>
  );
}
```

{% hint style="info" %}
Do **not** render `<OnboardModal />` yourself. `PrivacyBridgeProvider` mounts the modal once internally via `PrivateUnlockProvider`.
{% endhint %}

### Optional: network guard

Wrap page content with `NetworkGuard` to show a fallback UI when the wallet is on an unsupported chain:

```tsx
import { NetworkGuard } from '@coti-io/coti-wallet-plugin';

export function App() {
  return (
    <NetworkGuard fallback={<p>Please switch to a supported network.</p>}>
      <Dashboard />
    </NetworkGuard>
  );
}
```

## 2. Configure the plugin (before rendering hooks)

Call `configureCotiPlugin()` once at app startup, before any plugin hooks run:

```tsx
import { configureCotiPlugin } from '@coti-io/coti-wallet-plugin';

configureCotiPlugin({
  snapId: 'npm:@coti-io/coti-snap',
  aesKeyChainId: 7082400, // COTI Testnet
  walletConnectProjectId: 'your-project-id',
  debug: false,
});
```

See [Configuration](configuration.md) for all options.

## 3. Add an unlock control

Use `usePrivateUnlock()` for unlock orchestration in your UI:

```tsx
import { usePrivateUnlock } from '@coti-io/coti-wallet-plugin';

export function HeaderUnlockButton() {
  const privateUnlock = usePrivateUnlock();

  return (
    <button
      onClick={() => privateUnlock.toggleLock()}
      disabled={privateUnlock.isUnlocking}
    >
      {privateUnlock.isUnlocked ? 'Lock Private Balances' : 'Unlock Private Balances'}
    </button>
  );
}
```

### `usePrivateUnlock()` API

| Method / property | Description |
| --- | --- |
| `isUnlocked` | `true` when private balances are visible |
| `isUnlocking` | `true` while unlock/onboarding is in progress |
| `unlock()` | Start the unlock flow |
| `lock()` | Hide private balances |
| `toggleLock()` | Toggle between locked and unlocked |
| `requireUnlock(action)` | Run `action` after ensuring unlock; returns `true` if unlocked |
| `reset()` | Reset unlock UI state |

## 4. Guard private actions

Use `requireUnlock(action)` for any operation that needs private balance or key access. It tries the cached session key first, then backup/Snap restore, then the onboarding modal only if needed.

```tsx
import {
  usePrivateUnlock,
  usePrivacyBridgeUnlock,
} from '@coti-io/coti-wallet-plugin';

export function EncryptButton() {
  const privateUnlock = usePrivateUnlock();
  const unlock = usePrivacyBridgeUnlock();

  const encrypt = async () => {
    const result = await unlock.encryptPrivateValue({
      amount: '1.0',
      decimals: 18,
    });
    console.log(result.ciphertext);
  };

  return (
    <button
      disabled={privateUnlock.isUnlocking}
      onClick={() => void privateUnlock.requireUnlock(encrypt)}
    >
      Encrypt
    </button>
  );
}
```

## 5. Display token balances

Use the bounded context hooks to read wallet and token state:

```tsx
import {
  usePrivacyBridgeWallet,
  usePrivacyBridgeTokens,
} from '@coti-io/coti-wallet-plugin';

export function BalancePanel() {
  const { isConnected, walletAddress } = usePrivacyBridgeWallet();
  const { publicTokens, privateTokens } = usePrivacyBridgeTokens();

  if (!isConnected) return <p>Connect your wallet</p>;

  return (
    <div>
      <p>Address: {walletAddress}</p>
      <h3>Public tokens</h3>
      {publicTokens.map((t) => (
        <div key={t.symbol}>{t.symbol}: {t.balance}</div>
      ))}
      <h3>Private tokens</h3>
      {privateTokens.map((t) => (
        <div key={t.symbol}>{t.symbol}: {t.balance}</div>
      ))}
    </div>
  );
}
```

Private balances appear only after unlock. Public balances are always readable on-chain.

## 6. Bridge public ↔ private tokens

Use the swap context for portal operations:

```tsx
import { usePrivacyBridgeSwap } from '@coti-io/coti-wallet-plugin';

export function BridgeForm() {
  const {
    amount,
    setAmount,
    direction,
    setDirection,
    handleSwap,
    isBridgingLoading,
    estimatedGasFee,
  } = usePrivacyBridgeSwap();

  return (
    <div>
      <input value={amount} onChange={(e) => setAmount(e.target.value)} />
      <select
        value={direction}
        onChange={(e) => setDirection(e.target.value as 'to-private' | 'to-public')}
      >
        <option value="to-private">Portal In (public → private)</option>
        <option value="to-public">Portal Out (private → public)</option>
      </select>
      <button onClick={() => handleSwap()} disabled={isBridgingLoading}>
        {isBridgingLoading ? 'Bridging…' : 'Bridge'}
      </button>
      {estimatedGasFee && <p>Estimated gas: {estimatedGasFee}</p>}
    </div>
  );
}
```

On COTI chains, bridging uses the native COTI bridge. On Sepolia and Avalanche Fuji, bridging routes through the PoD Privacy Portal.

## Lock semantics

Understanding lock behavior is important for both UX and security:

```
lock()
  → balances hidden
  → session AES key kept in memory

unlock()
  → try cached session key
  → try restore backup / Snap
  → open onboarding modal only if restore fails
```

{% hint style="warning" %}
`isUnlocked` (or `isPrivateUnlocked`) means **private balances are visible** — it does not guarantee a key exists in Snap or backup storage. Do not use it to infer onboarding state.
{% endhint %}

Contract onboarding always ends on the plugin success screen. The user can reveal/copy the raw AES key, then click Done. Any pending action passed to `requireUnlock` runs after Done.

## Onboarding routes

| Wallet | First route | Fallback |
| --- | --- | --- |
| MetaMask with Snap | Retrieve AES key from Snap | Contract onboarding if Snap is empty |
| Other wallets / MetaMask without Snap | Encrypted backup restore | Contract onboarding, then manual AES key input |

See [Configuration](configuration.md) for encrypted backup and grant service setup, and [AES Key Onboarding](aes-key-onboarding.md) for the full contract onboarding flow.

## Do

* Use `PrivacyBridgeProvider privateUnlock={...}` once near app root.
* Use `usePrivateUnlock()` for unlock orchestration: `unlock()`, `lock()`, `toggleLock()`, `requireUnlock(action)`.
* Use `usePrivacyBridgeUnlock()` inside or after that guard for private operations: `sendPrivateToken`, `encryptPrivateValue`, `decryptPrivateValue`.
* Let the plugin own `OnboardModal`, Snap install, restore-only flow, contract onboarding, and the AES key success screen.

## Do not

* Do not render `OnboardModal` for unlock in app pages.
* Do not call `refreshPrivateBalances({ restoreOnly: true })` as a custom unlock flow.
* Do not infer key existence from `isPrivateUnlocked`; it only means private balances are visible.
* Do not delete Snap-stored keys or encrypted backups on lock. Lock only clears plaintext session state.

## Error handling

The plugin throws typed `CotiPluginError` instances with structured `CotiErrorCode` values. See [API Reference — Error codes](api-reference.md#error-codes) for the full list.

```tsx
import {
  CotiPluginError,
  CotiErrorCode,
  isCotiPluginError,
} from '@coti-io/coti-wallet-plugin';

try {
  await privateUnlock.unlock();
} catch (error) {
  if (isCotiPluginError(error)) {
    switch (error.code) {
      case CotiErrorCode.SNAP_CONNECT_FAILED:
        // Prompt Snap install
        break;
      case CotiErrorCode.USER_REJECTED:
        // User cancelled — no action needed
        break;
      case CotiErrorCode.AES_KEY_MISMATCH:
        // Prompt re-onboarding
        break;
    }
  }
}
```

## Related docs

* [Configuration](configuration.md)
* [API Reference](api-reference.md)
* [Onboard Modal Theming](onboard-modal-theme.md)
* [COTI MetaMask Snap Integration](../coti-metamask-snap/snap-integration.md) — low-level Snap RPC details
