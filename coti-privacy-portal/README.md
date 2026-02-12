# COTI PrivacyPortal

The COTI Privacy Portal allows you to convert your public ERC20 tokens into Private Tokens. These private tokens are stored securely via the COTI MetaMask Snap and can be transferred between COTI wallets with full privacy regarding transaction amounts.

## Key Features

The **COTI Portal Bridge** enables:

| Feature                           | Description                                                                                                                                    |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 🔒**Confidentiality as a Service** | Lock public assets and mint private, encrypted equivalents (e.g.,`USDC` → `p.USDC`)                                                            |
| 🛡️**Encrypted Operations**         | Private tokens can be held and transacted with complete privacy—amounts and balances remain encrypted on-chain, visible only to the key holder |
| 🔗**Seamless Integration**         | COTI Snaps bridge the gap between complex cryptography and usability, managing keys securely within MetaMask                                   |

## Supported Tokens

The Portal currently supports the following tokens:

| Public Token | Private Token | Decimals | Type   |
| ------------ | ------------- | -------- | ------ |
| **COTI**     | p.COTI        | 18       | Native |
| **WETH**     | p.WETH        | 18       | ERC20  |
| **WBTC**     | p.WBTC        | 8        | ERC20  |
| **USDT**     | p.USDT        | 6        | ERC20  |
| **USDC.e**   | p.USDC.e      | 6        | ERC20  |
| **WADA**     | p.WADA        | 18       | ERC20  |
| **gCOTI**    | p.gCOTI       | 18       | ERC20  |

## Prerequisites

Before you begin, ensure you have:

1.  **MetaMask installed in your browser.**
2.  **[COTI Network configured in MetaMask](../networks/mainnet/adding-the-coti-mainnet-to-metamask.md).** The Portal will prompt you to add it if needed.
3.  **The COTI Snap configured in MetaMask.** Please follow [COTI MetaMask Snap installation](https://dev.metamask.coti.io) instructions.
4.  **Some native COTI for transaction fees.**

## Guides

*   **[User Guide](user-guide.md)**: Step-by-step instructions for connecting, bridging, and managing tokens.
*   **[Troubleshooting & FAQ](troubleshooting.md)**: Solutions for common issues and answers to frequent questions.
*   **[Developer Tutorial](developer-guide.md)**: Architecture details, contract flows, and setup instructions.

