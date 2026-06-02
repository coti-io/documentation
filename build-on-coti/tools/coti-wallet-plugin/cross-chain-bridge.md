# Cross-Chain Bridge

These hooks enable token transfers between COTI and Ethereum networks. They are distinct from the [Privacy Bridge](privacy-bridge.md) hooks which handle public ↔ private token movement on the same COTI chain.

## `useCrossChainBridge()`

Executes cross-chain bridge transactions with pre-validation (limits, minimums, balance checks).

### Methods & Properties

```typescript
function bridgeNative(amount: bigint, tokenId: string): Promise<void>
```

Sends native token to the configured bridge recipient address.

```typescript
function bridgeERC20(amount: bigint, tokenId: string, tokenAddress: `0x${string}`): Promise<void>
```

Calls ERC20 `transfer()` to the configured bridge recipient.

* **`isLoading`** (`boolean`): Whether a bridge transaction is in progress.
* **`error`** (`BridgeError | null`): Typed error with codes: `DAILY_LIMIT_EXCEEDED`, `BELOW_MINIMUM`, `INSUFFICIENT_BALANCE`, `TRANSACTION_FAILED`, `UNSUPPORTED_TOKEN`.
* **`txHash`** (`string | null`): Transaction hash after successful submission.

---

## `useTransactionTracking(txHash, sourceNetworkId, destinationNetworkId)`

Polls the tracking service for real-time transaction progress.

* **`currentStep`** (`number | null`): Current stage (COTI-source: 4 steps, Ethereum-source: 3 steps).
* **`destinationHash`** (`string | null`): Destination chain tx hash when completed.
* **`failureReason`** (`string | null`): Reason if the transaction failed.
* **`failedStep`** (`number | null`): Step number where failure occurred.
* **`fee`** (`string | null`): Bridge fee from the tracking service.
* **`isLoading`** / **`error`**: Standard loading and error states.

---

## `useBridgeTransactions(walletAddress, pageSize, pageNumber)`

Fetches paginated transaction history with 30-second caching.

* **`transactions`** (`BridgeTransaction[]`): Enriched transaction records with current step, completion status, and destination hash.
* **`totalCount`** (`number`): Total number of transactions for pagination.
* **`isLoading`** / **`error`**: Standard loading and error states.

---

## `useBridgeLimits(walletAddress, tokenId)`

Polls the Cap Meter API for user and global daily bridge limits (default: every 30 seconds).

* **`userDailyLimit`** (`string`): User's remaining daily limit in human-readable token units.
* **`globalDailyLimit`** (`string`): Global remaining daily limit.
* **`isLoading`** / **`error`**: Standard loading and error states.

---

## `useWalletStatus()`

Reports wallet chain validity for cross-chain bridge operations and provides network switching.

* **`isConnected`** (`boolean`): Connection status.
* **`address`** (`string`): Connected address or empty string.
* **`chainId`** (`number | null`): Current chain ID.
* **`isValidChain`** (`boolean`): Whether the current chain is valid for bridge operations.
* **`switchChain(chainId)`** (`(chainId: number) => Promise<void>`): Switches to a valid chain.
* **`switchError`** (`string | null`): Error from last failed switch attempt.
* **`disconnect()`** (`() => void`): Disconnects the wallet.

---

## `useOngoingTransactions()`

Monitors all in-progress bridge operations via a module-level registry that persists across component mount/unmount.

* **`transactions`** (`OngoingTransaction[]`): All in-progress transactions with current step, destination hash, and failure info.

Use `registerTransaction({ tokenId, sourceChainId, destinationChainId, txHash })` to add a transaction to the registry.

---

## Utility Functions

| Function | Signature | Description |
| --- | --- | --- |
| `formatTokenAmount` | `(value: bigint, decimals: number) => string` | Formats bigint to decimal string without trailing zeros |
| `parseTokenAmount` | `(value: string, decimals: number) => bigint` | Parses decimal string to bigint (throws on invalid input) |
| `truncateDecimals` | `(value: string, maxDecimals: number) => string` | Truncates without rounding |
| `getActiveChains` | `(connectedChainId?) => ChainPair[]` | Returns valid chain pairs for the current environment |
| `isValidChain` | `(chainId, connectedChainId?) => boolean` | Checks if chain is valid for bridge operations |
| `getActiveChainById` | `(chainId, connectedChainId?) => ChainConfig \| undefined` | Returns chain config for a specific chain |
| `getActiveNetworks` | `(connectedChainId?) => ChainConfig[]` | Returns all active chain configs |
| `getCrossChainTokenConfig` | `(tokenId, chainId) => CrossChainTokenConfig \| undefined` | Returns token configuration for bridge operations |

---

## Usage

```tsx
import {
  useCrossChainBridge,
  useTransactionTracking,
  useWalletStatus,
  registerTransaction,
  parseTokenAmount,
} from '@coti-io/coti-wallet-plugin';

function CrossChainBridge() {
  const { bridgeNative, bridgeERC20, isLoading, error, txHash } = useCrossChainBridge();
  const { isValidChain, switchChain } = useWalletStatus();
  const tracking = useTransactionTracking(txHash, 7082400, 11155111);

  const handleBridgeNative = async () => {
    if (!isValidChain) {
      await switchChain(7082400); // Switch to COTI Testnet
    }

    const amount = parseTokenAmount('10', 18); // 10 COTI
    await bridgeNative(amount, 'COTI');

    // Register for ongoing monitoring
    if (txHash) {
      registerTransaction({
        tokenId: 'COTI',
        sourceChainId: 7082400,
        destinationChainId: 11155111,
        txHash,
      });
    }
  };

  return (
    <div>
      <button onClick={handleBridgeNative} disabled={isLoading}>
        {isLoading ? 'Bridging...' : 'Bridge 10 COTI to Ethereum'}
      </button>
      {error && <p>Error: {error.message}</p>}
      {tracking.currentStep && <p>Step {tracking.currentStep} of 4</p>}
      {tracking.destinationHash && <p>Done! Dest tx: {tracking.destinationHash}</p>}
    </div>
  );
}
```
