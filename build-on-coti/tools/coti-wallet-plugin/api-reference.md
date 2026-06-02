# API Reference

## Wallet Operations

### `useWallet()`

`useWallet()` is the recommended entry point for all wallet operations. It composes the lower-level hooks internally and manages the full wallet and AES key lifecycle.

#### Connection

* **`isConnected`** (`boolean`): Whether a wallet is currently connected.
* **`walletAddress`** (`string`): The connected wallet address.
* **`connect()`** (`() => Promise<void>`): Opens the wallet connection flow.
* **`disconnect()`** (`() => Promise<void>`): Revokes permissions and clears all state/caches.

#### Network

* **`networkName`** (`string`): Human-readable network name (e.g. "COTI Mainnet").
* **`chainId`** (`string | null`): Current chain ID as a decimal string.
* **`switchNetwork(chainId)`** (`(hex: string) => Promise<boolean>`): Requests the wallet to switch chains.
* **`COTI_MAINNET_ID`** (`string`): `"0x282b34"`
* **`COTI_TESTNET_ID`** (`string`): `"0x6c11a0"`
* **`SEPOLIA_ID`** (`string`): `"0xaa36a7"`

#### AES Key Lifecycle

* **`sessionAesKey`** (`string | null`): Current AES key (React state only, never persisted).
* **`isPrivateUnlocked`** (`boolean`): `true` when the session key is set.
* **`getAesKey(address)`** (`(addr: string) => Promise<string | null>`): Retrieves the AES key (routes to Snap for MetaMask, or Onboarding Contract for others).
* **`unlockPrivateBalances()`** (`() => Promise<boolean>`): Calls `getAesKey` for the current address and sets the session key.
* **`lockPrivateBalances()`** (`() => void`): Clears the session key and snap cache.
* **`clearKeyCache()`** (`() => void`): Forces a fresh retrieval on the next unlock.

---

### `usePrivateTokenBalance()`

Provides a unified interface to retrieve and decrypt private balances safely.

```typescript
function fetchPrivateBalance(
  userAddress: string,
  aesKey: string,
  contractAddress: string,
  version: number,
  decimals?: number
): Promise<string>
```

Fetches and decrypts the balance. Pass `64` for legacy native p.COTI, or `256` for wrapped/bridged private ERC20s.

* **Parameters:**
  * `userAddress`: The wallet address to query.
  * `aesKey`: The user's AES key for decryption.
  * `contractAddress`: The private token contract address.
  * `version`: `64` for legacy native p.COTI, `256` for bridged/private ERC20s.
  * `decimals`: Token decimals (optional, defaults to 18).
* **Returns:** The decrypted balance as a formatted string.

---

### `useBalanceUpdater(props)`

Advanced orchestrator typically used at the Provider level to manage global token states and batch-fetch the entire wallet portfolio in parallel.

```typescript
function updateAccountState(
  account: string,
  checkSnap?: boolean,
  fetchPrivate?: boolean,
  aesKeyOverride?: string,
  chainOverride?: number
): Promise<void>
```

Triggers a parallelized refresh of all configured COTI/ERC20 and p.ERC20 token balances.

---

### `useAesKeyProvider(walletTypeInfo)`

Routes AES key retrieval to Snap (MetaMask) or onboard contract (others).

* **`getAesKey(address)`** (`Promise<string>`): Resolves the AES key for the connected network/wallet type.
* **`isOnboarding`** (`boolean`): Indicates whether the onboarding process is actively running.
* **`onboardingError`** (`string | null`): Catches and exposes onboarding flow errors.

---

## Auxiliary Hooks & Utilities

* **`useBridgeData()`**: On-chain bridge state (fees, limits, paused status).
* **`useBridgeStatus()`**: Real-time bridge transaction status tracking.
* **`useNetworkEnforcer()`**: Enforces COTI-only networks, prompts chain switch.
* **`useSnap()` / `useMetamask()`**: Standalone hooks for legacy, pure-MetaMask implementations.
* **`formatTokenBalanceDisplay(balance)`**: Standardizes token display with thousand separators.
