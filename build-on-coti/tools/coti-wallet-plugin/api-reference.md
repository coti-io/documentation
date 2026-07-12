# API Reference

Public exports from `@coti-io/coti-wallet-plugin`.

## Providers

### `WagmiRainbowKitProvider`

Sets up wagmi v2, RainbowKit, and multi-wallet connectors (MetaMask EIP-6963, Rabby, OneKey, Zerion).

```tsx
<WagmiRainbowKitProvider walletConnectProjectId="your-project-id">
  {children}
</WagmiRainbowKitProvider>
```

Also exports `getWagmiConfig()`, `wagmiConfig`, and `WagmiConfigOptions`.

### `PrivacyBridgeProvider`

Core provider for wallet, network, tokens, unlock, swap, and PoD state. Nests `PrivateUnlockProvider` internally.

```tsx
<PrivacyBridgeProvider privateUnlock={{ theme, warning }}>
  {children}
</PrivacyBridgeProvider>
```

**Props:**

| Prop | Type | Description |
| --- | --- | --- |
| `children` | `React.ReactNode` | App content |
| `privateUnlock` | `PrivateUnlockProviderOptions` | Unlock modal theme, warning text, callbacks |

### `PrivateUnlockProvider`

Mounted internally by `PrivacyBridgeProvider`. Exposed for advanced use cases only.

## Hooks — unlock

### `usePrivateUnlock()`

High-level unlock controller. Preferred API for app UI.

| Property / method | Type | Description |
| --- | --- | --- |
| `isUnlocked` | `boolean` | Private balances are visible |
| `isUnlocking` | `boolean` | Unlock/onboarding in progress |
| `unlock()` | `() => Promise<void>` | Start unlock flow |
| `lock()` | `() => void` | Hide private balances |
| `toggleLock()` | `() => void` | Toggle lock state |
| `requireUnlock(action)` | `(action?) => Promise<boolean>` | Ensure unlock, then run action |
| `reset()` | `() => void` | Reset unlock UI state |

### `usePrivacyBridgeUnlock()`

Low-level unlock and private crypto operations. Use inside or after `requireUnlock`.

| Property / method | Type | Description |
| --- | --- | --- |
| `isPrivateUnlocked` | `boolean` | Private balances visible |
| `sessionAesKey` | `string \| null` | Session-bound AES key (avoid direct use) |
| `aesKeyChainId` | `number \| undefined` | COTI chain for AES state |
| `hasSnap` | `boolean` | Snap detected |
| `sendPrivateToken` | `(params) => Promise<{ txHash }>` | Send a private token transfer |
| `encryptPrivateValue` | `(params) => Promise<{ ciphertext }>` | Encrypt amount to ctUint256 JSON |
| `decryptPrivateValue` | `(params) => Promise<{ amount }>` | Decrypt ctUint256 JSON to amount |
| `lockPrivateBalances` | `() => void` | Hide balances, clear session key |
| `refreshPrivateBalances` | `(options?) => Promise<boolean>` | Low-level balance refresh (do not use for unlock UI) |

**`sendPrivateToken` params:**

```typescript
{ symbol: string; recipient: string; amount: string }
```

**`encryptPrivateValue` / `decryptPrivateValue` params:**

```typescript
{ amount: string; decimals?: number }  // encrypt
{ ciphertext: string; decimals?: number }  // decrypt
```

## Hooks — bounded context slices

Prefer these over the legacy flat `usePrivacyBridgeContext()` for new code.

### `usePrivacyBridgeWallet()`

| Property / method | Type | Description |
| --- | --- | --- |
| `isConnected` | `boolean` | Wallet connected |
| `walletAddress` | `string` | Connected address |
| `handleConnect` | `() => Promise<void>` | Open connect flow |
| `handleDisconnect` | `() => Promise<void>` | Disconnect wallet |
| `metamaskDetected` | `boolean` | MetaMask provider detected |

### `usePrivacyBridgeNetwork()`

| Property / method | Type | Description |
| --- | --- | --- |
| `chainId` | `string \| null` | Current chain ID |
| `switchNetwork` | `(chainId) => Promise<boolean>` | Switch wallet network |
| `networkName` | `string` | Human-readable network name |
| `isUnsupportedNetwork` | `boolean` | Chain not supported |
| `enforceNetwork` | `() => Promise<void>` | Prompt network switch |

### `usePrivacyBridgeTokens()`

| Property | Type | Description |
| --- | --- | --- |
| `publicTokens` | `Token[]` | Public token balances |
| `privateTokens` | `Token[]` | Private token balances (after unlock) |

### `usePrivacyBridgeSwap()`

| Property / method | Type | Description |
| --- | --- | --- |
| `amount` | `string` | Bridge amount input |
| `direction` | `'to-private' \| 'to-public'` | Bridge direction |
| `selectedTokenIndex` | `number` | Selected token index |
| `handleSwap` | `(amount?, direction?, tokenIndex?, onProgress?) => Promise<void>` | Execute bridge |
| `isBridgingLoading` | `boolean` | Bridge in progress |
| `isApprovalNeeded` | `boolean` | ERC20 approval required |
| `handleApprove` | `() => Promise<void>` | Approve ERC20 spend |
| `estimatedGasFee` | `string \| null` | Estimated gas fee display |
| `portalFeeCoti` | `string \| null` | COTI bridge portal fee |
| `portalFee` | `string \| null` | PoD portal fee (ETH/AVAX) |
| `isPodChain` | `boolean` | Connected chain uses PoD portal |

### `usePrivacyBridgePod()`

| Property / method | Type | Description |
| --- | --- | --- |
| `podRequests` | `PodPortalRequest[]` | Tracked PoD portal requests |
| `refreshPodRequest` | `(request) => Promise<void>` | Refresh request status |

### `usePrivacyBridgeModals()`

| Property | Type | Description |
| --- | --- | --- |
| `showInstallModal` | `boolean` | Snap install modal visible |
| `showMultipleWalletsModal` | `boolean` | Multiple wallets conflict modal |

### `usePrivacyBridgeContext()` (legacy)

Flat union of all slices. Existing consumers may continue using this; new code should prefer bounded hooks.

## Hooks — utilities

| Hook | Description |
| --- | --- |
| `useWalletType()` | Detect wallet type (MetaMask, Rabby, etc.) |
| `usePrivateTokenBalance()` | Fetch and decrypt a single private token balance |
| `useBalanceUpdater()` | Balance refresh utility |
| `useNetworkEnforcer()` | Network enforcement helper |
| `useMetamask()` | MetaMask provider helper |
| `useConnectModal()` | Re-exported from RainbowKit |

## Components

### `OnboardModal`

Onboarding UI component. **Do not render directly** for unlock — `PrivateUnlockProvider` mounts it.

Exports `onboardModalDefaultStyles` and `ONBOARD_MODAL_STYLE_KEYS` for theming. See [Onboard Modal Theming](onboard-modal-theme.md).

### `NetworkGuard`

Blocks children when the wallet is on an unsupported network.

```tsx
<NetworkGuard fallback={<UnsupportedNetwork />}>
  {children}
</NetworkGuard>
```

## Configuration

| Export | Description |
| --- | --- |
| `configureCotiPlugin(config)` | Set plugin configuration at startup |
| `getPluginConfig()` | Read current configuration |
| `getSnapRequestParams(snapId?, snapVersion?)` | Params for `wallet_requestSnaps` |
| `isSnapInstallEnabled()` | Whether Snap install is allowed |

See [Configuration](configuration.md) for all `CotiPluginConfig` options.

## Chain registry

| Export | Description |
| --- | --- |
| `CHAIN_CONFIGS` | All supported chain configurations |
| `getChainConfig(chainId)` | Lookup chain config |
| `getTokensForChain(chainId)` | Token list for a chain |
| `getUnlockStrategyForChain(chainId)` | Chain-config metadata (`snap` or `manual-aes-key`); not the runtime unlock order — see [AES Key Onboarding](aes-key-onboarding.md) |
| `cotiMainnet`, `cotiTestnet`, `sepolia` | viem chain definitions |
| `COTI_MAINNET_CHAIN_ID`, `COTI_TESTNET_CHAIN_ID`, `SEPOLIA_CHAIN_ID` | Chain ID constants |

## Contracts and tokens

| Export | Description |
| --- | --- |
| `CONTRACT_ADDRESSES` | Contract addresses per chain |
| `SUPPORTED_TOKENS` | Supported token configurations |
| `getPublicTokensForChain(chainId)` | Public tokens for a chain |
| `getPrivateTokensForChain(chainId)` | Private tokens for a chain |
| `TOKEN_ABI`, `BRIDGE_ABI`, `ERC20_ABI` | Contract ABIs |
| `LIMITS` | Bridge amount limits |

## Error handling

### `CotiPluginError`

```typescript
class CotiPluginError extends Error {
  readonly code: CotiErrorCode;
  readonly detail?: string;
}
```

### Type guards

```typescript
isCotiPluginError(error: unknown): error is CotiPluginError
hasCotiErrorCode(error: unknown, code: CotiErrorCode): boolean
```

### Error codes

#### Wallet / provider

| Code | Description |
| --- | --- |
| `METAMASK_NOT_INSTALLED` | MetaMask or EIP-1193 provider not installed |
| `NO_PROVIDER` | No EIP-1193 provider on `window.ethereum` |
| `USER_REJECTED` | User rejected wallet request (EIP-1193 code 4001) |

#### Snap

| Code | Description |
| --- | --- |
| `SNAP_CONNECT_FAILED` | Snap not installed or connection failed |
| `SNAP_DIALOG_REJECTED` | User dismissed Snap dialog |
| `SNAP_REQUIRED` | Snap required but unavailable for wallet type |
| `SNAP_KEY_CHECK_FAILED` | Snap key existence check failed |

#### AES key / onboarding

| Code | Description |
| --- | --- |
| `AES_KEY_MISMATCH` | AES key does not match on-chain account |
| `AES_KEY_MISSING` | AES key missing or not provided |
| `ACCOUNT_NOT_ONBOARDED` | Account never onboarded to COTI |
| `ONBOARDING_INCOMPLETE` | Onboarding did not complete |

#### Network

| Code | Description |
| --- | --- |
| `UNSUPPORTED_NETWORK` | Connected to unsupported chain |
| `WALLETCONNECT_PROJECT_ID_MISSING` | WalletConnect project ID not configured |

#### Bridge / transaction

| Code | Description |
| --- | --- |
| `INSUFFICIENT_BALANCE` | Insufficient token balance |
| `INSUFFICIENT_ALLOWANCE` | ERC20 allowance too low |
| `CONTRACT_NOT_FOUND` | Contract address not found for chain |
| `TRANSACTION_REVERTED` | On-chain transaction reverted |
| `ORACLE_TIMESTAMP_MISMATCH` | Stale oracle price data |

#### Other

| Code | Description |
| --- | --- |
| `API_ERROR` | External API returned non-success status |
| `VALIDATION_ERROR` | Input validation failed |

## Logging

| Export | Description |
| --- | --- |
| `logger` | Plugin logger instance |
| `setDebugLogging(enabled)` | Toggle debug logging |

Logging is silent by default. Enable via `configureCotiPlugin({ debug: true })`. Secrets are never logged.

## Utilities

| Export | Description |
| --- | --- |
| `formatTokenBalanceDisplay` | Format token balance for display |
| `truncateDecimalValue` | Truncate decimal values |
| `getEthereumProvider()` | Resolve EIP-1193 provider |
| `muteChainUpdates()` / `unmuteChainUpdates()` | Suppress UI during cross-chain onboarding |
| `isMultipleWalletsError(error)` | Detect multiple-wallet conflict |

## Related docs

* [Integration Guide](integration-guide.md)
* [Configuration](configuration.md)
* [Onboard Modal Theming](onboard-modal-theme.md)
