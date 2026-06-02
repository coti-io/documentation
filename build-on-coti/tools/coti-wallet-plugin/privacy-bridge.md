# Privacy Bridge

The Privacy Bridge hooks orchestrate Portal In (deposit) and Portal Out (withdraw) operations — moving tokens between their public and private representations on the same COTI chain.

## `usePrivacyBridge()`

Full bridge orchestration — deposit, withdraw, allowance, and fee estimation.

### Methods & Properties

```typescript
function handleSwap(
  amount?: string,
  direction?: 'to-private' | 'to-public',
  tokenIndex?: number,
  onProgress?: (status: string) => void
): Promise<void>
```

Unified method to execute a deposit (`'to-private'`) or withdraw (`'to-public'`).

* **Parameters:**
  * `amount`: The amount of tokens to bridge (as a decimal string).
  * `direction`: `'to-private'` for deposit, `'to-public'` for withdraw.
  * `tokenIndex`: Index of the selected token in the configured token array.
  * `onProgress`: Optional callback for progress status updates.

---

* **`isBridgingLoading`** (`boolean`): Indicates a swap/bridge transaction is in progress.
* **`isApprovalNeeded`** (`boolean`): Indicates whether the selected swap requires an ERC20 allowance approval.

```typescript
function handleApprove(): Promise<void>
```

Execute the approval allowance transaction for a bridge out operation.

* **`estimatedGasFee`** (`string`): Active gas fee estimation for the selected bridge route.
* **`portalFeeCoti`** (`string`): Portal fee in COTI for the selected bridge route.

## Usage

```tsx
import { usePrivacyBridge } from '@coti-io/coti-wallet-plugin';

function BridgeComponent() {
  const { handleSwap, isBridgingLoading } = usePrivacyBridge();

  const handleDeposit = async () => {
    // 100 tokens, sending to private side, token array index 0
    await handleSwap('100', 'to-private', 0);
  };

  return (
    <button onClick={handleDeposit} disabled={isBridgingLoading}>
      {isBridgingLoading ? 'Depositing...' : 'Deposit to Private'}
    </button>
  );
}
```
