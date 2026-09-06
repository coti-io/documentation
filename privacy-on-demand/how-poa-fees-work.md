# How do PoA fees work?

**PoA fees** here are the **Privacy on Demand (PoD)** fees you pay in **native token** on your EVM chain (for example Sepolia **ETH**) when you use a **two-way Inbox** message: your contract sends **`add(itUint64 a, itUint64 b)`**, and the success path returns **`ctUint64 c`** through a **callback**. The same mechanics apply to other `PodLib` operations that use **`msg.value`** and **`callbackFeeLocalWei`**.

This page explains how that payment is **split**, converted (via **oracles**) into **execution budgets** on each side—often described as **gas units**—and **consumed** step by step.

The numbers below are a **single worked example** so you can follow the arithmetic. Live networks use **oracle and Inbox policy** to set conversion rates and minimums; use your deployment’s **views** (for example `calculateTwoWayFeeRequiredInLocalToken`) and the SDK’s [Fees, gas, and oracle](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md) reference for production.

## Example call

Solidity shape (conceptually):

- **Request:** `add(itUint64 a, itUint64 b)` with **`msg.value`** and **`callbackFeeLocalWei`**
- **Success callback:** receives **`abi.encode(ctUint64 c)`** (encrypted sum)

## Tables: fee flow and consumption

**Serialized flow:** ledgers open after the oracle; **COTI** spends first (relay + private add), then **Sepolia** (relay response + callback). **`sendTwoWayMessage`** binds the payment to outbound + return routes. Exact splits and minima are enforced on-chain (**Inbox**, **fee manager**).

### From user fee to gas-unit ledgers (single table)

| Stage | Amounts |
| --- | --- |
| **User-provided gas** | <ul><li><strong>0.007 ETH</strong> (<code>msg.value</code>)</li><li><strong>0.0035 ETH</strong> (<code>callbackFeeLocalWei</code>, Sepolia)</li><li><strong>0.0035 ETH</strong> (COTI-side)</li><li><strong>5 gwei</strong> (reference)</li></ul> |
| **Price at oracle** (same quote, e.g. USD) | <ul><li><strong>COTI:</strong> <strong>0.017</strong> per 1 COTI.</li><li><strong>ETH:</strong> <strong>2,170</strong> per 1 ETH.</li></ul> |
| **Converted to the chain gas (COTI — ETH)** | <ul><li><strong>COTI:</strong> <strong>446.76 COTI</strong></li><li><strong>ETH:</strong> <strong>0.0035 ETH</strong></li></ul> |
| **Convert to gas units (COTI — ETH)** | <ul><li><strong>COTI:</strong> <strong>16,029,096</strong></li><li><strong>ETH:</strong> <strong>615,701</strong></li></ul> |

### Gas used and remaining (both legs)

| Step | Where | What happens | Gas used (step) | COTI remaining | Sepolia remaining |
| --- | --- | --- | --- | --- | --- |
| — | — | Balances after oracle | — | **16,029,096** | **615,701** |
| 1 | COTI | Relayers relay request | **12,000** | **16,017,096** | **615,701** |
| 2 | COTI | Private **add** → ciphertext **c** | **492,902** | **15,524,194** | **615,701** |
| 3 | Sepolia | Miner relays response | **8,000** | **15,524,194** | **607,000** |
| 4 | Sepolia | **`addCallback`** stores **`ctUint64 c`** | **230,000** | **15,524,194** | **377,000** |

## Walkthrough

Read the **first table top to bottom:** user ETH is split by leg, oracles supply **COTI** and **ETH** prices, the COTI leg is expressed as **quote → COTI tokens**, then policy turns each leg into **gas-unit budgets**. The **second table** spends **COTI** first, then **Sepolia** after the result exists. Underspend remains are illustrative; production behavior depends on **InboxMiner** / **InboxFeeManager** and operator policy (see the SDK [Fees, gas, and oracle](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md) doc).

## Why this matters for `add(a, b) → ctUint64 c`

- **Private execution** and **callback execution** happen in **different environments**, as **separate steps in time**: COTI runs **first**; the Sepolia relay and **callback** run **after** the result exists. Each leg needs its **own** paid work (compute + relay).  
- **`callbackFeeLocalWei` is structurally important**: if the **callback leg** is under-funded, **`c` may never be written**, and UIs stay **pending** (see [Async private operations](async-private-operations.md)).  
- **Too little `msg.value`** tends to hurt the **outbound / COTI** path; **too little callback slice** tends to hurt the **return / Sepolia** path.

## Where to implement this in code

- Solidity: payable **`add`** with **`msg.value`** and **`callbackFeeLocalWei`**, as in [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) (see [Tutorials overview](tutorials-privacy-on-demand.md) for how this fits the **primitive-only** model).  
- Estimation: Inbox **`calculateTwoWayFeeRequiredInLocalToken`** and the [Fees, gas, and oracle](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md) document in the PoD SDK repo.

## Disclaimer

The **rates**, **unit counts**, and **remainders** on this page are **pedagogical**. They **do not** replace reading **live** on-chain configuration, **oracles**, and **Inbox / miner** behavior for your network.
