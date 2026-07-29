# How do PoA fees work on Avalanche Fuji?

**PoA fees** are the **Privacy on Demand** fees you pay in **AVAX** on **Avalanche Fuji C-Chain** (`43113`) for a **two-way Inbox** message. Your Fuji contract typically calls something like **`add(itUint64 a, itUint64 b)`**, and the success path returns **`ctUint64 c`** in a later **callback** on Fuji. The same payment model applies to other `PodLib` operations that take **`msg.value`** and **`callbackFeeLocalWei`**.

You always pay on Fuji in AVAX. Oracles convert the **COTI-bound slice** into a remote execution budget; the **callback slice** stays denominated for Fuji gas.

> **ABI note:** The parameter is named `callbackFeeLocalWei` in Solidity. On Fuji that means the **18-decimal AVAX subunit** (same decimal layout as wei on Ethereum), not ETH. Avalanche gas prices are usually quoted in **nAVAX** (\(10^{-9}\) AVAX = \(10^{9}\) of those subunits).

## What you are paying for

A two-way PoD request does work on **two chains, in order**:

1. **COTI (outbound leg)** — relay the request and run private compute (for example MPC `add`).
2. **Avalanche Fuji (return leg)** — relay the response and run your **`onlyInbox` callback** so `c` is written on Fuji.

If either leg is underfunded, the job can stall as **pending** even when the other leg would have succeeded (see [Async private operations](async-private-operations.md)).

`callbackFeeLocalWei` is a **slice of** `msg.value`, not an extra payment on top.

## How Fuji AVAX becomes gas-unit budgets

The Inbox fee manager (conceptually) does this when you call `sendTwoWayMessage` / a `PodLib` helper:

| Symbol | Meaning on Fuji |
| --- | --- |
| \(V\) | `msg.value` in AVAX subunits |
| \(C\) | `callbackFeeLocalWei` (Fuji / callback slice), \(C \le V\) |
| \(R\) | Remote slice \(R = V - C\) (funds COTI via oracle) |
| \(G\) | Effective gas price of the Fuji tx (subunits per gas), typically tens of **nAVAX** |

**Fuji callback budget (gas units):**

\[
\texttt{callerFee} = \left\lfloor \frac{C}{G} \right\rfloor
\]

**COTI remote budget (gas units), after oracle scaling:**

\[
\texttt{targetFee} = \left\lfloor \frac{R}{G} \right\rfloor \times \frac{P_{\mathrm{AVAX}}}{P_{\mathrm{COTI}}}
\]

where \(P_{\mathrm{AVAX}}\) and \(P_{\mathrm{COTI}}\) are the Fuji Inbox **price oracle** USD prices for local (AVAX) and remote (COTI) tokens.

Because AVAX is typically much more valuable than COTI per token, a modest AVAX remote slice often converts into a **large** COTI gas-unit budget. That is expected on Fuji; do not copy Sepolia ETH numbers.

Live conversions and minima come from the deployed Fuji Inbox / fee manager / oracle — always prefer on-chain views over this page’s arithmetic.

## Worked example (Avalanche Fuji, pedagogical)

Assumptions for one readable walkthrough (not live oracle quotes):

| Input | Value |
| --- | --- |
| `msg.value` | **0.02 AVAX** |
| `callbackFeeLocalWei` | **0.008 AVAX** (Fuji callback slice) |
| Remote slice \(R\) | **0.012 AVAX** (pays for COTI leg) |
| Fuji gas price \(G\) | **25 nAVAX** |
| Oracle \(P_{\mathrm{AVAX}}\) | **\$25** / AVAX |
| Oracle \(P_{\mathrm{COTI}}\) | **\$0.017** / COTI |

### Step A — Split the AVAX payment

| Leg | AVAX | Role |
| --- | --- | --- |
| Callback (Fuji) | 0.008 | Relayer response + `addCallback` on C-Chain |
| Remote (COTI) | 0.012 | Relay + private `add` on COTI |
| **Total** | **0.02** | Attached as `msg.value` on the Fuji tx |

USD value of the remote slice: \(0.012 \times 25 = \$0.30\), or about **17.65 COTI** at \$0.017 / COTI. That intuition check is optional; the Inbox stores **gas-unit** budgets, not a COTI balance in your wallet.

### Step B — Convert to gas-unit ledgers

**Fuji `callerFee`:**

\[
\frac{0.008\ \mathrm{AVAX}}{25\ \mathrm{nAVAX}} = 320{,}000\ \text{gas units}
\]

**Pre-oracle remote units** (AVAX paid ÷ Fuji gas price):

\[
\frac{0.012\ \mathrm{AVAX}}{25\ \mathrm{nAVAX}} = 480{,}000
\]

**COTI `targetFee`** (scale by AVAX/COTI oracle prices):

\[
480{,}000 \times \frac{25}{0.017} \approx 705{,}882{,}353\ \text{gas units}
\]

| Ledger | Opening balance (gas units) |
| --- | --- |
| COTI (`targetFee`) | **705,882,353** |
| Avalanche Fuji (`callerFee`) | **320,000** |

### Step C — Spend the ledgers in order

COTI runs first; Fuji callback runs only after the encrypted result exists.

| Step | Where | What happens | Gas used | COTI remaining | Fuji remaining |
| --- | --- | --- | --- | --- | --- |
| — | — | Opening balances | — | **705,882,353** | **320,000** |
| 1 | COTI | Relayers relay request | **12,000** | **705,870,353** | **320,000** |
| 2 | COTI | Private **add** → ciphertext **c** | **492,902** | **705,377,451** | **320,000** |
| 3 | Avalanche Fuji | Miner relays response | **8,000** | **705,377,451** | **312,000** |
| 4 | Avalanche Fuji | **`addCallback`** stores **`ctUint64 c`** | **230,000** | **705,377,451** | **82,000** |

Remainders are illustrative. Production refunds / keep policies depend on **InboxMiner** / **InboxFeeManager** configuration.

## Why underfunding fails differently on each leg

- **Too little `msg.value` overall** (or too little remote slice) → COTI relay / private execution may not clear minima → outbound path fails.
- **`callbackFeeLocalWei` too small** → COTI may finish, but Fuji never successfully writes **`c`** → UI stays **pending**.
- **Gas price matters:** budgets are \({\approx}\,\mathrm{fee}/G\). A higher Fuji `tx.gasprice` for the same AVAX payment yields a **smaller** gas-unit budget. Estimate with the gas price you will actually use.

## How to budget fees in practice on Fuji

1. Prefer the deployed Fuji Inbox view:

   `calculateTwoWayFeeRequiredInLocalToken(...)`

2. Or estimate from TypeScript with `@coti-io/pod-sdk`:

```typescript
const fee = await pod.estimateFee("add", podArgs, {
  forwardGasLimit: 400_000n,
  callBackGasLimit: 250_000n,
  callBackDataSize: 512n,
  gasPrice: (await signer.provider!.getFeeData()).gasPrice!,
});
// fee.totalFee → msg.value
// fee.callBackFee → callbackFeeLocalWei (must be ≤ totalFee)
```

3. Attach `fee.totalFee` as `msg.value` and pass `fee.callBackFee` as `callbackFeeLocalWei` (mark that argument with `isCallBackFee: true` when using `PodContract`).

4. Confirm the Fuji price oracle and Inbox addresses on [Avalanche Fuji](networks/fuji.md).

## Where this shows up in Solidity

- Payable **`add`** (or other `PodLib` helpers) with **`msg.value`** and **`callbackFeeLocalWei`** — see [Tutorial: private Adder on Avalanche Fuji](tutorial-private-adder-fuji.md).
- Integration model context: [Tutorials overview](tutorials-privacy-on-avalanche.md).
- Contract-level detail: SDK [Fees, gas, and oracle](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/04-fees-gas-and-oracle.md).

## Disclaimer

Oracle prices, gas prices, minimum fee templates, and remainders on this page are **pedagogical**. They show the **Fuji AVAX → dual gas-unit ledger** story with arithmetic that matches the Inbox fee-manager shape. They **do not** replace live Fuji Inbox / oracle configuration for your deployment.
