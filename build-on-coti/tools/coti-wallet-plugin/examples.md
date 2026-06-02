# Usage Examples

## Basic Setup (MetaMask Only)

```tsx
import { configureCotiPlugin, PrivacyBridgeProvider } from '@coti-io/coti-wallet-plugin';

// Optional — configure before rendering (defaults work for mainnet)
configureCotiPlugin({
  snapId: 'npm:@coti-io/coti-snap',
  defaultNetworkId: '0x282b34', // COTI Mainnet (2632500)
});

function App() {
  return (
    <PrivacyBridgeProvider>
      <YourApp />
    </PrivacyBridgeProvider>
  );
}
```

## Any Wallet Setup (RainbowKit + wagmi)

```tsx
import { WagmiRainbowKitProvider, PrivacyBridgeProvider } from '@coti-io/coti-wallet-plugin';

function App() {
  return (
    <WagmiRainbowKitProvider walletConnectProjectId="your-project-id">
      <PrivacyBridgeProvider>
        <YourApp />
      </PrivacyBridgeProvider>
    </WagmiRainbowKitProvider>
  );
}
```

## Fetch and Decrypt a Private Balance

```tsx
import { usePrivateTokenBalance } from '@coti-io/coti-wallet-plugin';

function PrivateBalanceViewer({ userAddress, aesKey, tokenAddress }) {
  const { fetchPrivateBalance } = usePrivateTokenBalance();

  const handleFetch = async () => {
    try {
      // Pass 64 for legacy native p.COTI, or 256 for bridged/private ERC20s
      const balance = await fetchPrivateBalance(userAddress, aesKey, tokenAddress, 256, 18);
      console.log(`Decrypted Balance: ${balance}`);
    } catch (error) {
      console.error("Failed to decrypt:", error.message);
    }
  };

  return <button onClick={handleFetch}>Fetch Balance</button>;
}
```

## Use the Privacy Bridge Hook

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

## Cross-Chain Bridge (COTI ↔ Ethereum)

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

## Example App

A complete working example is available in the [`examples/`](https://github.com/coti-io/coti-wallet-plugin/tree/main/examples) directory. It demonstrates wallet connection, public ERC20 balance reading, and private balance decryption using tokens from the [COTI Token List](https://github.com/coti-io/coti-token-list).

### Prerequisites

* Node.js 18+
* The parent plugin must be built first

### Setup & Run

```bash
# 1. Build the plugin (from the repository root)
npm run build

# 2. Move into the examples directory
cd examples

# 3. Copy the environment template and add your WalletConnect project ID
cp .env.example .env

# 4. Install dependencies
npm install

# 5. Start the dev server
npm run dev
```

Opens at http://localhost:5173

{% hint style="info" %}
Get a WalletConnect project ID at [https://cloud.walletconnect.com](https://cloud.walletconnect.com) and set it in `examples/.env`:

```
VITE_WALLETCONNECT_PROJECT_ID=your_project_id_here
```
{% endhint %}

### What the Example App Does

1. **Connect Wallet** — Click to open the RainbowKit modal (MetaMask, Coinbase, Rabby, WalletConnect, etc.)
2. **Public Balances** — Reads on-chain ERC20 `balanceOf` for all public tokens on COTI Testnet
3. **Native COTI** — Displays native COTI balance via wagmi
4. **Private Balances** — Click "Unlock Private Balances" to derive the AES key (via Snap for MetaMask, or on-chain onboarding contract for other wallets), then decrypted private token balances appear

### Network

The example app targets **COTI Testnet** (chain ID 7082400). Ensure your wallet has COTI Testnet funds. On unlock, the app automatically prompts a network switch if needed.
