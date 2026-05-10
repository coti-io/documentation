# User guide

The Privacy Portal is COTI V2's user-facing `dApp` for interacting with tokens on COTI, letting you bridge between Public tokens and [PrivateERC20](../developer-guide/privateerc20.sol.md) tokens.

It allows you to work with two types of tokens:

## Public Tokens

Public Tokens behave like standard blockchain tokens:

* balances are visible on-chain
* transactions are transparent

## Private Tokens

Private Tokens add an extra layer of privacy:

* balances are stored encrypted on-chain
* you decrypt and view them locally using your wallet and the COTI MetaMask Snap

This helps keep your financial data confidential.

## What you’ll learn

This guide will walk you through how to:

* connect your wallet to the Privacy Portal
* bridge public tokens into private tokens
* send and receive private transactions
* view your balances securely

## Bridge status (Portal behavior)

The Privacy Portal should **read on-chain state** for the selected bridge **before** submitting transactions: **`paused()`** (full stop), **`isDepositEnabled`** (deposits off), and **`blacklisted(address)`** for the connected wallet. That way users see a **clear message** instead of an opaque wallet revert (`Pausable: paused`, `DepositDisabled`, `AddressBlacklisted`, etc.).

## Protocol fee vs network gas

**Bridge / Portal fee (COTI):** Dynamic fee paid to the protocol (in **native COTI** on ERC‑20 flows, or taken from the amount on native COTI flows), later swept to the configured **`feeRecipient`**. This is **not** the same as **network gas** (paid to validators for including the transaction). Support and docs should use distinct labels so users do not conflate the two line items.

**Percentage fee ceiling:** On-chain operators cannot set the **percentage** component above **10%** of the fee divisor (`MAX_FEE_UNITS` in the bridge contracts) without a **new deployment**. Public messaging should not promise a higher **percentage leg** than that **hard ceiling**; **fixed** and **max** fee knobs in COTI wei are separate levers.

## Why use the Privacy Portal?

The Privacy Portal lets you move your tokens into a private state, giving you more control over what information is visible on-chain while keeping sensitive data confidential.

## Terms and on-chain risk (summary for legal / ToS)

Using the **privacy bridges** (via the Portal or any integrator) should be described in plain language as acceptance of:

1. **Oracle-based fees** — Bridge fees use **Band Protocol** prices read through the deployed **`CotiPriceConsumer`** (and each bridge’s **`priceOracle`** pointer). Users accept **oracle correctness**, **update cadence**, and **staleness rules** documented in the [developer architecture](../developer-guide/architecture.md) and [troubleshooting](../developer-guide/troubleshooting.md) pages.
2. **Immutable payout addresses** — **`feeRecipient`** and **`rescueRecipient`** are **fixed at deploy** (no on-chain setter). Users accept where **protocol fees** and **emergency-rescued** funds may be sent.
3. **Governance powers** — An **`owner`** (ideally multisig) and **`OPERATOR_ROLE`** addresses can **pause**, **toggle deposits**, **set limits**, **rotate the oracle pointer**, **update fee parameters**, **blacklist addresses**, and (when paused) **rescue** custodied assets to **`rescueRecipient`**. These powers are **on-chain**; link **[contract addresses](../developer-guide/contract-addresses.md)** and explorers so users can verify **who** holds those roles for each deployment.

Product **Terms of Service** should not bury the above only inside generic “DeFi risks”; point readers to **this section** and the linked technical pages.

## Official explorers (anti-phishing)

User-facing docs and the Privacy Portal should link **only** the **official** block explorer base URLs for each network (e.g. Cotiscan / Blockscout as configured for COTI mainnet and testnet). Do not paste generic “etherscan-style” links that could mislead users to **wrong** sites when verifying bridge or token addresses.

## Confirmations and finality (sequencer networks)

COTI V2 uses a **sequencer**, so a transaction can appear **included** very quickly in the UI and explorer. That is **not** the same as **strong finality** for high-value decisions (reorgs or rare replays are still an EVM-class concern). Product copy and support should recommend waiting roughly **5–10 blocks** after inclusion before treating a **Portal In / Out** as fully settled for **large** amounts or downstream actions (e.g. moving funds elsewhere), unless your risk team publishes a different number.
