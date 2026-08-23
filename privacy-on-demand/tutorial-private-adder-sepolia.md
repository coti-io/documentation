# Tutorial: private Adder on Ethereum Sepolia

This walkthrough is the **primitive-only** path: your host-chain contract calls **`PodLib`** helpers (the SDK surface for **MpcLib**-style primitives) and never deploys custom Solidity on COTI. If you are unsure whether that is enough for your product, read **[Tutorials: building Privacy on Demand (PoD) dApps](tutorials-privacy-on-demand.md)** first.

This guide shows how to build a minimal **Privacy on Demand** dApp that **adds two encrypted integers** on COTI and stores the **encrypted sum** on your EVM contract. It follows the same ideas as the SDK’s [MpcAdder.sol](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/MpcAdder.sol) example, extended with **Sepolia routing presets** and **request correlation** suitable for a real UI.

For background on async flows and fees, see [Async private operations](async-private-operations.md), [How do PoA fees work?](how-poa-fees-work.md), and the SDK’s [Fees, gas, and oracle](https://github.com/coti-io/coti-sdk-pod/tree/main/site/contracts/04-fees-gas-and-oracle) page.

## Writing a PoD example

In this example we will do the following:

1. **Use one built-in executor operation** from `PodLib` (the sample below uses **`add256`**) so you do not write a custom COTI contract yet.
2. **Submit a payable request** that forwards `msg.value` and splits out `callbackFeeLocalWei` for the return leg (two-way Inbox message).
3. **Implement a success callback** that decodes `abi.encode(ctUint256)` and stores the ciphertext.
4. **Wire `onDefaultMpcError.selector`** so failed remote runs surface through the SDK’s default error path (and emit `ErrorRemoteCall` from `PodUser`).

After that works, you harden for production: per-user request ownership, explicit `pending / completed / failed` state, fee estimation via the Inbox, and tests for under-funded sends. The SDK’s [Examples with description](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05c-examples-with-description) lists what the shipped `MpcAdder` omits on purpose.

## Prerequisites

- **Solidity toolchain** (Foundry or Hardhat) targeting **Ethereum Sepolia** (where the SDK’s `PodUserSepolia` Inbox is deployed).
- **Node.js 18+** for scripts and `fetch` used by encryption helpers.
- **Sepolia ETH** for deployment and for **`msg.value`** on each `add` call (plus gas).
- **User onboarding** so your client can obtain an **account AES key** for decryption (see the SDK’s [TypeScript integration](https://github.com/coti-io/coti-sdk-pod/tree/main/site/06-typescript-integration-ux-development) and [Onboarding / account AES key](https://github.com/coti-io/coti-sdk-pod/tree/main/site/06c-onboarding-account-account-aes-key) docs).

Always confirm **Inbox**, **COTI chain id**, and **MPC executor** against `PodUserSepolia.sol` / `PodNetworkConstants.sol` in your installed `@coti-io/coti-contracts` package (and [Ethereum Sepolia](networks/sepolia.md)); constants can change between releases.

## Step 1: Install packages

```bash
# TypeScript helpers (encrypt / fees / send)
npm install @coti-io/pod-sdk ethers

# Solidity (PodLib, PodUserSepolia, MpcCore) — currently install from GitHub main
npm install github:coti-io/coti-contracts#main
```

`@coti-io/pod-sdk` is **TypeScript only**. Solidity imports come from `@coti-io/coti-contracts`.
## Step 2: Create the `PrivateAdder` contract

Save as `PrivateAdder.sol`. The contract:

- Inherits **`PodLib`** and **`PodUserSepolia`** (Sepolia defaults for Inbox and COTI routing).
- Calls **`add256`** with the caller’s encrypted inputs and your callback selector.
- Resolves **`requestId`** in the callback the same way as the SDK’s [Getting started](https://github.com/coti-io/coti-sdk-pod/tree/main/site/04-getting-started) example.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.26;

import "@coti-io/coti-contracts/contracts/pod/mpc/PodLib.sol";
import "@coti-io/coti-contracts/contracts/pod/mpc/PodUserSepolia.sol";
import "@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol";

/// @title PrivateAdder
/// @notice Adds two encrypted uint64 values via PoD on Sepolia (SDK preset addresses).
contract PrivateAdder is PodLib, PodUserSepolia {
    enum RequestStatus {
        None,
        Pending,
        Completed
    }

    mapping(bytes32 => ctUint256) public sumByRequest;
    mapping(bytes32 => RequestStatus) public statusByRequest;

    event AddRequested(bytes32 indexed requestId, address indexed caller);
    event AddCompleted(bytes32 indexed requestId);

    constructor() PodLibBase(msg.sender) {
        setInbox(INBOX_ADDRESS);
        configureCoti(MPC_EXECUTOR_ADDRESS, COTI_CHAIN_ID);
    }

    /// @param callbackFeeLocalWei Wei reserved for the callback leg; must be <= msg.value (see SDK fee docs).
    function add(
        itUint256 calldata a,
        itUint256 calldata b,
        uint256 callbackFeeLocalWei
    ) external payable returns (bytes32 requestId) {
        requestId = add256( // Add two encrypted 256 bit integers
            a,
            b,
            msg.sender, // Who can decrypt the result
            this.addCallback.selector,
            this.onDefaultMpcError.selector,
            msg.value,
            callbackFeeLocalWei
        );
        statusByRequest[requestId] = RequestStatus.Pending;
        emit AddRequested(requestId, msg.sender);
    }

    function addCallback(bytes memory data) external onlyInbox {
        bytes32 requestId = inbox.inboxSourceRequestId();
        if (requestId == bytes32(0)) {
            requestId = inbox.inboxRequestId();
        }

        ctUint256 memory sum = abi.decode(data, (ctUint256));
        sumByRequest[requestId] = sum;
        statusByRequest[requestId] = RequestStatus.Completed;
        emit AddCompleted(requestId);
    }
}
```

**Notes:**

- **`onDefaultMpcError`** is implemented on `PodLibBase` and forwards failures to **`ErrorRemoteCall`** on `PodUser`. Your UI can listen for that event to mark a request failed.
- **`addCallback`** must stay **`onlyInbox`** so random accounts cannot forge results.
- **`ctUint256`** is a Solidity **struct** `{ ctUint128 ciphertextHigh; ctUint128 ciphertextLow; }`, so the decoded local must use a `memory` location and the storage mapping holds the two‑limb tuple. The narrower garbled / ciphertext types (`gtUint8…gtUint256`, `gtBool`, and `ctUint8…ctUint128`) are **user‑defined value types** — pass and assign them like `uint256` (no `memory` / `calldata`). Encrypted-input wrappers such as **`itUint256`** stay structs and keep their `calldata` / `memory` location as before.

## Step 3: Compile and deploy on Sepolia

Configure remappings so `@coti-io/coti-contracts` resolves (Hardhat `paths`, Foundry `remappings.txt`, etc.), then compile and deploy `PrivateAdder` to **Ethereum Sepolia**. Record the deployed address for scripts.

## Step 4: Budget `msg.value` and `callbackFeeLocalWei`

Two-way Inbox traffic needs enough native token to cover **outbound execution** and the **callback**. Use the deployed Inbox’s **`calculateTwoWayFeeRequiredInLocalToken`** (or your operator’s runbook) to pick `msg.value` and `callbackFeeLocalWei`. Undershooting typically leaves the request stuck or failing.

## Step 5: Encrypt the two summands (TypeScript)

`CotiPodCrypto.encrypt` calls the PoD encryption service. For Sepolia-style test usage, pass **`"testnet"`** as the network key (see [`coti-pod-crypto.ts`](https://github.com/coti-io/coti-sdk-pod/blob/main/src/coti-pod-crypto.ts) in the SDK: `testnet` maps to the COTI testnet encryption endpoint).

Use **`DataType.itUint256`** when you build **`itUint256`** calldata yourself (for example with **`ethers.Contract`**). If you use **`PodContract.encryptAndCallMethod`** in the next step, you can skip manual encryption: pass **plaintext numeric strings** and **`DataType.itUint256`** in each `PodMethodArgument`, and the SDK encrypts before encoding the transaction.

```typescript
import { CotiPodCrypto, DataType } from "@coti-io/pod-sdk";

const plainA = "10";
const plainB = "20";

const encA = await CotiPodCrypto.encrypt(plainA, "testnet", DataType.itUint256);
const encB = await CotiPodCrypto.encrypt(plainB, "testnet", DataType.itUint256);

```

## Step 6: Submit the `add` transaction (`PodContract`, fees, `extractRequestIds`)

[`PodContract`](https://github.com/coti-io/coti-sdk-pod/blob/main/src/pod-method-call.ts) wraps your **`ethers.Contract`**: it **`estimateFee`**s against the Inbox, maps **`PodMethodArgument`** values (including **`encryptAndCallMethod`** encryption for **`it*`** types), injects the **`callBackFee`** into the slot marked **`isCallBackFee: true`**, sends **`value: totalFee`** on payable functions, and exposes **`extractRequestIds(txHash)`** to read **`requestId`** values from **`MessageSent`** logs on the Inbox (reliable across layouts where parsing logs from the app contract alone is brittle).

```typescript
import {
  PodContract,
  DataType,
  type PodFeeEstimationConfig,
  type PodMethodArgument,
} from "@coti-io/pod-sdk";
import { ethers } from "ethers";

// Minimal ABI fragment — prefer the full artifact from your build (Hardhat / Foundry).
// itUint256 = { ctUint256 ciphertext; bytes signature }, and ctUint256 = { uint256 ciphertextHigh; uint256 ciphertextLow }
// so each `itUint256` parameter encodes as ((uint256,uint256),bytes).
const privateAdderAbi = [
  "function add(((uint256,uint256),bytes),((uint256,uint256),bytes),uint256) payable returns (bytes32)",
] as const;

const pod = new PodContract(
  privateAdderAddress,
  privateAdderAbi,
  signer,
  { encryptionNetwork: "testnet" }
);

const args: PodMethodArgument[] = [
  { type: DataType.itUint256, value: "10", isCallBackFee: false },
  { type: DataType.itUint256, value: "20", isCallBackFee: false },
  { type: DataType.Uint256, value: "0", isCallBackFee: true },
];

const feeCfg: PodFeeEstimationConfig = {
  forwardGasLimit: 400_000n,
  gasPrice: (await signer.provider!.getFeeData()).gasPrice ?? 0n,
  callBackGasLimit: 250_000n,
  callBackDataSize: 512n,
};

const estimated = await pod.estimateFee("add", args, feeCfg);
// estimated.totalFee — forward + callback legs in local wei (see SDK fee docs)

const txResponse = await pod.encryptAndCallMethod("add", args, feeCfg);
const receipt = await (txResponse as ethers.ContractTransactionResponse).wait();

const requestIds = receipt?.hash ? await pod.extractRequestIds(receipt.hash) : [];
const requestId = requestIds[0];
// requestId is 0x-prefixed bytes32 — keep it for status polling and decryption
```

Tune **`forwardGasLimit`**, **`callBackGasLimit`**, and **`callBackDataSize`** from measurements on your contract; **`estimateFee`** requires **`callBackGasLimit`** and **`callBackDataSize`** together or neither. For **`callMethod`** instead of **`encryptAndCallMethod`**, supply **JSON ciphertext strings** for **`it*`** arguments (what the encryption service returns), for example when the browser already called **`CotiPodCrypto.encrypt`**.

**Alternative (raw `ethers.Contract`)** — If you are not using **`PodContract`**, call **`add`** with **`toitUint256(encA)`**, **`toitUint256(encB)`**, **`callbackFeeLocalWei`**, and **`{ value: totalWei }`**, then still use **`extractRequestIds`** for correlation:

```typescript
const podRead = new PodContract(
  privateAdderAddress,
  privateAdderAbi,
  signer,
  { encryptionNetwork: "testnet" }
);

const tx = await privateAdder.add(
  toitUint256(encA),
  toitUint256(encB),
  callbackFeeWei,
  { value: totalWei }
);
const receipt = await tx.wait();
const requestIds = receipt?.hash ? await podRead.extractRequestIds(receipt.hash) : [];
```

Private addition is **asynchronous**: the sum appears only after the Inbox invokes **`addCallback`** in a later transaction. Poll **`statusByRequest(requestId)`** or wait for **`AddCompleted`**.

## Step 7: Read the encrypted sum and decrypt locally

After status is **Completed**, read **`sumByRequest(requestId)`**. The value is **`ctUint256`** (ciphertext), not plaintext. Because **`ctUint256`** is a Solidity **struct** with two `ctUint128` limbs, the contract read returns a tuple `{ ciphertextHigh, ciphertextLow }` (each is a single `uint256`).

```typescript
import { CotiPodCrypto, DataType } from "@coti-io/pod-sdk";

// accountAesKey: hex string from your app’s COTI onboarding flow (never log it)

const raw = await privateAdder.sumByRequest(requestId);
// ethers / viem return the struct as a tuple — normalize to { ciphertextHigh, ciphertextLow }
const ct = {
  ciphertextHigh: BigInt(raw.ciphertextHigh ?? raw[0]),
  ciphertextLow:  BigInt(raw.ciphertextLow  ?? raw[1]),
};

const decryptedString = CotiPodCrypto.decrypt(
  ct,
  accountAesKey,
  DataType.Uint256
);

console.log("sum (plaintext string):", decryptedString);
// Expect "30" for plainA=10 and plainB=20
```

`CotiPodCrypto.decrypt` delegates to **`@coti-io/coti-sdk-typescript`** (`^1.0.7`), which now exposes `decryptUint256({ ciphertextHigh, ciphertextLow }, accountAesKey)` for the 256‑bit lane. Narrower lanes (`Uint64`, `Uint128`, …) still take a single `uint256` ciphertext as a `bigint` or `0x`‑prefixed hex string (see SDK source [coti-pod-crypto.ts](https://github.com/coti-io/coti-sdk-pod/blob/main/src/coti-pod-crypto.ts)).

## Step 8: Sanity checks and next steps

- **Callback decode** must stay **`(ctUint256)`** — and the local must use `memory` because `ctUint256` is a struct. Changing the executor op or COTI-side behavior without updating the decode tuple will corrupt storage reads.
- **Type lane** — This contract uses **`add256`** with **`itUint256`** / **`ctUint256`** on chain, so pass **`DataType.Uint256`** to `CotiPodCrypto.decrypt` and feed it the **`{ ciphertextHigh, ciphertextLow }`** tuple read from the contract. Narrower lanes (`Uint64`, `Uint128`) still take a single ciphertext word.
- **Type model** — In the current `MpcCore.sol`, `gtUint*`, `gtBool`, and `ctUint8…ctUint128` are **user‑defined value types** (`type X is uint256`) — drop `memory` / `calldata` on them. `ctUint256` is a struct (two `ctUint128` limbs); `itUint*` / `utUint*` are also still structs, so keep `calldata` / `memory` on those.
- **Production**: add tests for non-Inbox callers on `addCallback`, under-funded `msg.value`, and decrypt failures; follow the [first production checklist](https://github.com/coti-io/coti-sdk-pod/tree/main/site/04-getting-started) in Getting started.

## Reference links

- [`pod-method-call.ts` (`PodContract`, fees, `extractRequestIds`)](https://github.com/coti-io/coti-sdk-pod/blob/main/src/pod-method-call.ts)
- [MpcAdder.sol (minimal repo example)](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/MpcAdder.sol)
- [Examples with description](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05c-examples-with-description)
- [Getting started (PodUserSepolia pattern)](https://github.com/coti-io/coti-sdk-pod/tree/main/site/04-getting-started)
- [Async execution](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05a-async-execution)

<div style="width:100%; box-sizing:border-box; margin:2rem 0 0 0; padding:1.35rem 1rem; border:2px solid #334155; border-radius:10px; background:#f8fafc; text-align:center;">

<p style="margin:0.4rem 0; font-size:1.25rem; font-weight:700;"><a href="tutorial-private-adder-sepolia.md" style="color:#0f172a; text-decoration:none;">Tutorial: private Adder on Sepolia</a></p>

<p style="margin:0.4rem 0; font-size:1.25rem; font-weight:700;"><a href="tutorial-custom-logic.md" style="color:#0f172a; text-decoration:none;">Tutorial: custom privacy logic with PoD</a></p>

</div>
