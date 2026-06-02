# COTI Wallet Plugin

The [**COTI Wallet Plugin**](https://github.com/coti-io/coti-wallet-plugin) is a privacy enhancement layer for existing dApps and wallets. It is not a standalone wallet application — it is designed to be injected into your existing React/wagmi stack to seamlessly add COTI network privacy capabilities to any EIP-1193 compatible wallet.

## Overview

By hooking into standard EIP-1193 connections using **wagmi v2 and RainbowKit** as the underlying connection infrastructure, the plugin transparently adds COTI privacy features (AES key derivation, balance decryption, confidential transfers) to whatever wallet the user prefers to use.

### Key Capabilities

* **AES Key Management** — Retrieves encryption keys via MetaMask Snap or the COTI Onboarding Contract (multi-wallet support via RainbowKit + wagmi v2)
* **Balance Decryption** — Fetches encrypted on-chain balances and decrypts them client-side
* **Privacy Bridge** — Orchestrates Portal In (deposit) and Portal Out (withdraw) operations with fee estimation
* **Cross-Chain Bridge** — Transfers tokens between COTI and Ethereum networks (native and ERC20) with transaction tracking, limits management, and ongoing transaction monitoring
* **Network Configuration** — COTI Mainnet, Testnet, and Ethereum Mainnet chain definitions ready for wagmi/viem

### Supported Wallets

The plugin works with any EIP-1193 browser wallet via RainbowKit, including:

* MetaMask
* Coinbase Wallet
* Trust Wallet
* Rainbow
* WalletConnect
* Safe
* Argent
* Ledger Live
* Brave Wallet
* Kraken Wallet
* Phantom (EVM)
* OKX Wallet
* Zerion
* TokenPocket
* Bitget Wallet
* Any injected EIP-1193 browser wallet

### How It Works

When a user connects through RainbowKit, the plugin detects the wallet type via wagmi's stable `connector.id`. For MetaMask, it routes AES key retrieval through the COTI Snap. For all other wallets, it wraps the wallet's EIP-1193 provider into a `@coti-io/coti-ethers` BrowserProvider, obtains a signer, and calls `generateOrRecoverAes()` on the COTI Onboarding Contract — which prompts the user for a single signature to derive or recover their encryption key.

This means any wallet that supports standard message signing can participate in COTI's privacy features without needing a custom extension or snap.

## Installation

```bash
npm install @coti-io/coti-wallet-plugin
```

### Peer Dependencies

```bash
npm install react ethers viem @coti-io/coti-sdk-typescript @metamask/providers @rainbow-me/rainbowkit wagmi @tanstack/react-query
```

### Build Commands

```bash
npm run build    # Produces dist/index.js (CJS) + dist/index.mjs (ESM) + dist/index.d.ts
npm run lint     # TypeScript type check (tsc --noEmit)
npm run test     # Run test suite (vitest)
npm run clean    # Remove dist/
```

## Supported Networks

| Network           | Chain ID   | RPC                                          |
| ----------------- | ---------- | -------------------------------------------- |
| COTI Mainnet      | 2632500    | https://mainnet.coti.io/rpc                  |
| COTI Testnet      | 7082400    | https://testnet.coti.io/rpc                  |
| Ethereum Mainnet  | 1          | https://eth.llamarpc.com                     |
| Ethereum Sepolia  | 11155111   | https://ethereum-sepolia-rpc.publicnode.com   |

### Cross-Chain Bridge Chain Pairs

| Environment | COTI Network             | Ethereum Network       |
| ----------- | ------------------------ | ---------------------- |
| Testnet     | COTI Testnet (7082400)   | Sepolia (11155111)     |
| Mainnet     | COTI Mainnet (2632500)   | Ethereum Mainnet (1)   |


