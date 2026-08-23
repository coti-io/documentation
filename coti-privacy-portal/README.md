# COTI Privacy Portal

## Introduction

As the decentralized web evolves, the need for financial privacy has become increasingly important.

Many blockchain networks are fully transparent, meaning:

* transaction histories are public
* wallet balances are visible
* sensitive financial activity can be exposed

COTI V2 addresses this limitation by introducing an EVM-compatible Layer 2 network that enables computation on encrypted data, powered by advanced cryptography such as Garbled Circuits.

The Privacy Portal is the primary web interface for interacting with this system. It allows users to manage private assets, perform confidential transactions, and keep control over their financial data.

## 🧭 What is the Privacy Portal?

> **Native COTI Privacy Portal** — This section covers the **on-COTI** Privacy Portal and `PrivateERC20` flows (mint/burn/transfer entirely on COTI). It is **not** the **PoD cross-chain Privacy Portal** (factory + pTokens on Fuji/Sepolia that route private compute through the Inbox). For host-chain PoD portal addresses, see [Avalanche Fuji](../privacy-on-demand/networks/fuji.md) and [Ethereum Sepolia](../privacy-on-demand/networks/sepolia.md).

The Privacy Portal is COTI V2's dApp for interacting with privacy-enabled tokens. It lets users bridge supported ERC20 tokens into private tokens, view and use encrypted balances, and submit confidential transfers while balances and transfer amounts remain encrypted on-chain, with decryption handled locally through the COTI MetaMask Snap.

## ✨ What you can do with the Privacy Portal?

With the Privacy Portal, you can:

* **🪙 Mint Private Tokens**\
  Convert supported ERC20 tokens into Private Tokens, allowing you to use them within COTI's privacy layer.
* **🔐 Manage Private Balances**\
  View and manage your encrypted balances securely using the COTI MetaMask Snap.
* **🔄 Send Confidential Transactions**\
  Transfer assets between wallets while keeping transaction amounts private.
* **👁️ View Your Data Privately**\
  Decrypt and view your balances locally. Only you have access to your financial data.
* **🧾 Keep Balances Encrypted On-Chain**\
  All balances remain encrypted on the ledger and are only accessible to the authorized key holder.

## ⚙️ How does it work?

COTI enables computation on encrypted data, meaning your data remains private even while it is being used.

The Privacy Portal acts as a bridge between you and this system:

* You convert assets into Private Tokens
* Your balances are stored encrypted on-chain
* Transactions are executed without revealing sensitive data
* Your data is decrypted locally when you need to view it

This allows you to interact with blockchain-based assets while preserving strong privacy for balances and transaction amounts.

## 🛡️ Why COTI V2 for Privacy?

COTI V2 introduces a new approach to blockchain privacy by combining encryption, secure computation, and developer-friendly infrastructure.

### 🔒 Encrypted Token Balances

Unlike standard ERC20 tokens, where balances and transactions are public, COTI's `PrivateERC20` stores all balances and amounts as encrypted data (ciphertexts).

### 🧮 Secure Computation (Garbled Circuits and MPC)

COTI uses advanced cryptographic techniques such as Multi-Party Computation (MPC) and Garbled Circuits to process encrypted data.

Through the [`MpcCore`](../how-coti-works/advanced-topics/precompiles.md) precompile (address `0x64`), the network can perform operations like:

* addition
* subtraction
* comparisons

All without revealing the underlying data to validators or external observers.

### 🔑 Local Decryption

Your private balances are decrypted locally in your browser, using a personal AES key managed by the [COTI Snap](../build-on-coti/guides/setting-up-coti-snap-with-your-metamask-wallet.md). Decryption keys are managed locally by you.

### 🧩 EVM Compatibility

COTI V2 is fully EVM-compatible, allowing developers to build using familiar tools such as:

* Solidity
* Hardhat

Developers can extend the [`PrivateERC20`](developer-guide/privateerc20.sol.md) contract to create privacy-enabled tokens without learning a new stack.
