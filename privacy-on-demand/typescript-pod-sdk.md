# TypeScript PoD SDK (`CotiPodCrypto`, `PodContract`)

The npm package **`@coti-io/pod-sdk`** ships **TypeScript** helpers for **encrypting and decrypting** PoD payloads and for **encoding, fee estimation, and sending** calls against your host-chain contract through [`coti-pod-crypto.ts`](https://github.com/coti-io/coti-sdk-pod/blob/main/src/coti-pod-crypto.ts) and [`pod-method-call.ts`](https://github.com/coti-io/coti-sdk-pod/blob/main/src/pod-method-call.ts). Solidity (`PodLib`, `PodUserSepolia`, `MpcCore`) lives in **`@coti-io/coti-contracts`**, not in this package.

Use these helpers from a wallet script, backend service, or dApp frontend once you have a **`Signer`** (or **`Provider`** for read-only helpers) and your contract **ABI**.

## Encrypt and decrypt data (`CotiPodCrypto`)

`CotiPodCrypto.encrypt` POSTs to the PoD encryption service:

- `"testnet"` and `"mainnet"` resolve to SDK defaults.
- You can also pass a full encryption service URL.
- `DataType` distinguishes plaintext types (`Uint64`, `String`, etc.) from **`it*`** types that become ciphertext tuples on chain.

`CotiPodCrypto.decrypt` uses the user's **account AES key** and **`@coti-io/coti-sdk-typescript`** (`^1.0.7`) under the hood.

```typescript
import { CotiPodCrypto, DataType } from "@coti-io/pod-sdk";

// Encrypt plaintext for Solidity itUint256 parameters
const enc = await CotiPodCrypto.encrypt("42", "testnet", DataType.itUint256);

// Decrypt a narrow scalar ciphertext (one uint256 word) read from contract storage
const plain64 = CotiPodCrypto.decrypt("0x...", accountAesKeyFromOnboarding, DataType.Uint64);

// Decrypt a ctUint256 value (a struct with two ctUint128 limbs)
const plain256 = CotiPodCrypto.decrypt(
  { ciphertextHigh, ciphertextLow },
  accountAesKeyFromOnboarding,
  DataType.Uint256
);
```

### Ciphertext shape in the current `MpcCore.sol`

After the gt‑type upgrade, the on‑chain types you read back have these shapes:

| Type                       | Solidity                                                  | Off‑chain shape                                |
| -------------------------- | --------------------------------------------------------- | ---------------------------------------------- |
| `gtUint8` … `gtUint256`, `gtBool` | User‑defined value type (`type … is uint256`), no `memory`/`calldata` | n/a — never crosses the chain boundary         |
| `ctUint8` … `ctUint128`    | User‑defined value type                                   | single `uint256` word                          |
| `ctUint256`                | Struct `{ ctUint128 ciphertextHigh; ctUint128 ciphertextLow; }` | tuple `{ ciphertextHigh, ciphertextLow }` (each `bigint`) |
| `itUint*` / `utUint*`      | Struct (ciphertext + signature, unchanged)                | tuple / object                                 |
| `gtString` / `ctString`    | Struct (array of `gtUint64` / `ctUint64`, unchanged)      | array of words                                 |

When you read a `ctUint256` value via ethers or viem, the storage getter returns the two limbs as a tuple — normalize it to `{ ciphertextHigh, ciphertextLow }` before passing it to `CotiPodCrypto.decrypt` (or to `decryptUint256` from `@coti-io/coti-sdk-typescript`). The narrower `ct*` lanes can still be passed as a `bigint` or `0x`‑prefixed hex string.

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
} from "@coti-io/pod-sdk";
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
- [TypeScript integration (SDK docs)](https://github.com/coti-io/coti-sdk-pod/tree/main/site/06-typescript-integration-ux-development)
- [PoD SDK docs index](https://github.com/coti-io/coti-sdk-pod/tree/main/site)
