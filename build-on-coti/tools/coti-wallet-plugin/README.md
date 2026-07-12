---
description: React library for adding COTI private token support to wagmi/RainbowKit dApps.
---

# COTI Wallet Plugin

The **COTI Wallet Plugin** (`@coti-io/coti-wallet-plugin`) is a React library that lets dApp developers add COTI private token (pToken) support to an existing **wagmi v2 + RainbowKit** stack — without building a custom wallet or hand-rolling AES key management.

{% hint style="info" %}
**Important:** This library is a **plugin for existing dApps and wallets**, not a standalone wallet application. It is designed to be injected into your existing React/wagmi stack to seamlessly enhance standard wallets with COTI network privacy capabilities.
{% endhint %}

## When to use it

Use the COTI Wallet Plugin when you are building a **React dApp** that needs:

* Wallet connection via MetaMask, Rabby, WalletConnect, and other EIP-1193 wallets
* AES key onboarding and unlock for private token balances
* Public ↔ private token bridging
* Private encrypt/decrypt/send operations

If you are building without React or wagmi, use the lower-level [TypeScript SDK](../typescript-sdk.md), [Ethers.js](../ethers.js.md), or [COTI MetaMask Snap](../coti-metamask-snap/snap-integration.md) integration guides instead.

## What it does

| Capability | How |
| --- | --- |
| Wallet connection | `WagmiRainbowKitProvider` — MetaMask, Rabby, WalletConnect, and more |
| AES onboarding / unlock | Built-in `OnboardModal` — Snap, contract onboarding, encrypted backup restore |
| Private balance display | Auto-decrypt on-chain ciphertext balances |
| Public ↔ private bridging | Native COTI bridge on COTI chains; PoD Privacy Portal on Sepolia / Avalanche Fuji |
| Private crypto ops | `encryptPrivateValue`, `decryptPrivateValue`, `sendPrivateToken` |
| Network enforcement | `NetworkGuard` for supported chains |

## Supported chains

| Chain | Chain ID | Portal strategy |
| --- | --- | --- |
| COTI Testnet | 7082400 | COTI native bridge |
| COTI Mainnet | 2632500 | COTI native bridge |
| Sepolia | 11155111 | PoD Privacy Portal |
| Avalanche Fuji | 43113 | PoD Privacy Portal |

Unlock is wallet-based (same on every supported chain), in preferred order:

1. MetaMask Snap (when MetaMask + Snap is available)
2. Encrypted backup restore
3. Contract onboarding (on COTI)
4. Manual AES key input (if the host enables it)

See [AES Key Onboarding](aes-key-onboarding.md) for routes and fallbacks.

## Relationship to other COTI tools

| Product | Role |
| --- | --- |
| [COTI MetaMask Snap](../coti-metamask-snap/README.md) | Key storage and Snap-backed crypto; the plugin calls it via RPC |
| [TypeScript SDK](../typescript-sdk.md) | Low-level encrypt/decrypt primitives; wrapped internally by the plugin |
| [Ethers.js](../ethers.js.md) | Contract onboarding (`generateOrRecoverAes`); used internally |
| [COTI Privacy Portal](../../../coti-privacy-portal/README.md) | A dApp that may consume the plugin; not required to use the library |

## Installation

```bash
npm install @coti-io/coti-wallet-plugin
```

### Peer dependencies

```bash
npm install react react-dom ethers viem @coti-io/coti-sdk-typescript @metamask/providers @rainbow-me/rainbowkit wagmi @tanstack/react-query
```

{% hint style="warning" %}
This release is validated with `@rainbow-me/rainbowkit@2.2.0` and `wagmi@2.14.0`. Keep those two packages on compatible versions in the host app; installing mismatched latest peer versions can break RainbowKit before the plugin loads.
{% endhint %}

## Quickstart

```tsx
import {
  PrivacyBridgeProvider,
  WagmiRainbowKitProvider,
  usePrivateUnlock,
} from '@coti-io/coti-wallet-plugin';

export function Root() {
  return (
    <WagmiRainbowKitProvider walletConnectProjectId={process.env.WALLETCONNECT_PROJECT_ID}>
      <PrivacyBridgeProvider>
        <App />
      </PrivacyBridgeProvider>
    </WagmiRainbowKitProvider>
  );
}

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

See the [Integration Guide](integration-guide.md) for the full provider setup, unlock flow, and private action guards.

## dApp API contract

dApps should **not** handle AES keys in the normal integration path. Use the plugin provider and hook APIs to connect wallets, onboard/unlock private balances, and execute private operations. The plugin owns AES retrieval, backup restore, Snap storage, and Snap-backed decrypt/encrypt calls internally.

For MetaMask Snap wallets, runtime private operations use Snap RPCs without extracting the AES key from Snap. For non-Snap wallets, the plugin keeps any recovered key in plugin session state only as needed.

## Example app

See [Example App](example-app.md) for setup and run instructions. Source code lives in the [coti-wallet-plugin GitHub repository](https://github.com/coti-io/coti-wallet-plugin/tree/main/examples).

## Next steps

* [Integration Guide](integration-guide.md) — provider setup, unlock flow, lock semantics
* [Configuration](configuration.md) — `configureCotiPlugin()` and onboarding services
* [API Reference](api-reference.md) — hooks, providers, types, and error codes
* [AES Key Onboarding](aes-key-onboarding.md) — onboarding routes, contract flow, security
* [Onboard Modal Theming](onboard-modal-theme.md) — customize the onboarding UI
* [Example App](example-app.md) — runnable reference dApp
