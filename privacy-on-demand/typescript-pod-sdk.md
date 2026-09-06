# TypeScript PoD SDK (`CotiPodCrypto`, `PodContract`)

The npm package **`@coti/pod-sdk`** ships TypeScript helpers for **encrypting and decrypting** PoD payloads and for **encoding, fee estimation, and sending** calls against your host-chain contract through [`coti-pod-crypto.ts`](https://github.com/cotitech-io/coti-pod-sdk/blob/main/src/coti-pod-crypto.ts) and [`pod-method-call.ts`](https://github.com/cotitech-io/coti-pod-sdk/blob/main/src/pod-method-call.ts).

Use these helpers from a wallet script, backend service, or dApp frontend once you have a **`Signer`** (or **`Provider`** for read-only helpers) and your contract **ABI**.

## Encrypt and decrypt data (`CotiPodCrypto`)

`CotiPodCrypto.encrypt` POSTs to the PoD encryption service:

- `"testnet"` and `"mainnet"` resolve to SDK defaults.
- You can also pass a full encryption service URL.
- `DataType` distinguishes plaintext types (`Uint64`, `String`, etc.) from **`it*`** types that become ciphertext tuples on chain.

`CotiPodCrypto.decrypt` uses the user's **account AES key** and `@coti-io/coti-sdk-typescript` under the hood.

```typescript
import { CotiPodCrypto, DataType } from "@coti/pod-sdk";

// Encrypt plaintext for Solidity itUint256 parameters
const enc = await CotiPodCrypto.encrypt("42", "testnet", DataType.itUint256);

// Decrypt scalar ciphertext read from contract storage
const plain = CotiPodCrypto.decrypt("0x...", accountAesKeyFromOnboarding, DataType.Uint64);
```

## Gas estimation and method calls (`PodContract`)

`PodContract` wraps `ethers.Contract` with PoD-aware helpers:

- `estimateFee` for Inbox two-way fee calculation.
- `encryptAndCallMethod` to encrypt `it*` plaintext args and send the tx.
- `callMethod` to send pre-encrypted JSON ciphertext values.
- `extractRequestIds(txHash)` to parse Inbox `MessageSent` logs and recover request IDs for async tracking.

```typescript
import {
  PodContract,
  DataType,
  type PodFeeEstimationConfig,
  type PodMethodArgument,
} from "@coti/pod-sdk";
import { ethers } from "ethers";

const pod = new PodContract(contractAddress, abi, signer, {
  encryptionNetwork: "testnet",
});

const args: PodMethodArgument[] = [
  { type: DataType.itUint256, value: "10", isCallBackFee: false },
  { type: DataType.itUint256, value: "20", isCallBackFee: false },
  { type: DataType.Uint256, value: "0", isCallBackFee: true }, // auto-replaced with callback fee
];

const feeCfg: PodFeeEstimationConfig = {
  forwardGasLimit: 400_000n,
  gasPrice: (await signer.provider!.getFeeData()).gasPrice ?? 0n,
  callBackGasLimit: 250_000n,
  callBackDataSize: 512n,
};

const fee = await pod.estimateFee("add", args, feeCfg);
// fee.totalFee, fee.remoteFee, fee.callBackFee

const txResponse = await pod.encryptAndCallMethod("add", args, feeCfg);
const receipt = await (txResponse as ethers.ContractTransactionResponse).wait();

const requestIds = receipt?.hash ? await pod.extractRequestIds(receipt.hash) : [];
```

### Important fee config rules

- `forwardGasLimit` and `gasPrice` are required.
- `callBackGasLimit` and `callBackDataSize` must be provided together (or both omitted).
- `forwardDataSize` is optional; SDK can estimate from argument string sizes.

## Choosing `encryptAndCallMethod` vs `callMethod`

- Use **`encryptAndCallMethod`** when your app currently has plaintext user input.
- Use **`callMethod`** when the app already has ciphertext JSON from a prior `CotiPodCrypto.encrypt` step and you want to submit it directly.

## See also

- [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) — full walkthrough including `PodContract` and `extractRequestIds`.
- [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md) — custom COTI-side pattern.
- [TypeScript integration (SDK docs)](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/06-typescript-integration-ux-development.md)
- [PoD SDK docs index](https://github.com/cotitech-io/coti-pod-sdk/tree/main/docs)
