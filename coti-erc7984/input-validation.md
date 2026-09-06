# Input validation

Confidential transfers need a way to prove that an encrypted input is well-formed and bound to the caller. The stacks do that in different places.

On **COTI**, the contract (or the PoD path into COTI) calls an **on-chain precompile**. Invalid ciphertext or signature reverts in that transaction.

On **Zama FHEVM**, the client builds a zero-knowledge proof of knowledge (ZKPoK) locally, but **that proof is not the host-chain calldata**. The client sends the packed ciphertext and ZKPoK to the **Relayer**. Coprocessors verify it on the Gateway path and return **ECDSA-signed handles**. The contract’s `inputProof` argument is those signatures, checked by `InputVerifier`.

The practical consequences are Relayer/Gateway dependency, ZKPoK verification cost, and failures that can happen **before** any host-chain transaction is sent — not a multi-kilobyte ZK proof sitting in L1 calldata.

## Zama: ZKPoK off-chain, signed handles on-chain

Flow (current protocol, measured 6 September 2026):

1. The client encrypts under the FHE public key and produces a ZKPoK (WASM; `@zama-fhe/sdk` 3.5.1 / TFHE WASM v1.6.2). Multi-thread WASM needs `SharedArrayBuffer` and COOP/COEP; otherwise the SDK falls back to single-thread proving.
2. The SDK `POST`s the blob to the Relayer (`/v2/input-proof`). Coprocessors verify the ZKPoK and store the ciphertext.
3. The Relayer returns handles plus coprocessor signatures. That byte string is what Solidity calls `inputProof`.
4. The user (or the SDK) sends the host-chain transaction. `FHE.fromExternal(handle, inputProof)` checks signatures against a threshold in `InputVerifier`. Comments in that contract state the layout: `numHandles + numSigners + handles + coprocessorSignatures` (+ `extraData`).

If Relayer or coprocessor verification fails, **no host transaction is submitted**. Users still pay for ZKPoK verification on the Gateway path (Relayer-coordinated). Both **input registration** and **decryption** depend on that Relayer / Gateway / KMS flow.

### What we measured

Encrypting one `euint64` for Sepolia [cUSDCMock](https://sepolia.etherscan.io/address/0x7c5BF43B851c1dff1a4feE8dB225b87f2C223639) via `https://relayer.testnet.zama.org`:

| Object | Size | What it is |
| :----- | ---: | :--------- |
| Relayer field `ciphertextWithInputVerification` | **18,794 bytes** | Packed ciphertext + ZKPoK (hex payload, no `0x` prefix; 37,588 hex chars) |
| Host-chain `inputProof` returned by `sdk.encrypt` | **230 bytes** | `0x01` handles, `0x03` signers, 32-byte handle, 3 × 65-byte signatures, 1-byte `extraData` |
| Live `confidentialTransfer` calldata (9 txs, 5 Sep 2026) | **452 bytes** typical | Selector + `address` + handle + ABI-encoded 230-byte `inputProof` |

Example host transaction: [`0x1a8b317a…d2976b`](https://sepolia.etherscan.io/tx/0x1a8b317ad2593aacad3ad1a9344482e7eeac2cea6d288729fcef02052ad2976b) (`inputProof` length 230, gas used 1,464,996).

`InputVerifier` documents the same split: the off-chain bundle is `compressedPackedCT+ZKPOK`; the handle is derived from it; calldata is handles and coprocessor signatures.

Illustrative TypeScript ([Zama encrypt & decrypt](https://docs.zama.org/protocol/sdk/guides/encrypt-decrypt)):

```typescript
import { ZamaSDK } from "@zama-fhe/sdk";

// Client WASM builds a ZKPoK. sdk.encrypt() sends it to the Relayer and
// waits for coprocessor signatures. Failures here happen before any host tx.
const { encryptedValues, inputProof } = await sdk.encrypt({
  values: [{ value: amount, type: "euint64" }],
  contractAddress,
  userAddress,
});

// inputProof is signed handles, not the ZKPoK.
await token.confidentialTransfer(to, encryptedValues[0], inputProof);
```

Legacy Hardhat plugin shape (`createEncryptedInput(...).encrypt()`) is the same architecture: prove locally, register via Relayer/Gateway, pass attestations on-chain.

## COTI: `validateCiphertext` on-chain

The user encrypts with their AES key and signs the input (`it*`: ciphertext + signature). The contract (or PoD path into COTI) calls the MPC precompile. If the ciphertext or signature is invalid, the call **reverts**. There is no client ZK proof of encryption validity and no Relayer registration step before the host transaction.

Precompile address: `0x0000000000000000000000000000000000000064`.

Solidity (via `MpcCore`):

```solidity
import {MpcCore, itUint256, gtUint256} from "@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol";

function transfer(address to, itUint256 calldata value) external {
    // On-chain validation: bad ciphertext / signature → revert.
    gtUint256 gtValue = MpcCore.validateCiphertext(value);
    _transfer(msg.sender, to, gtValue);
}
```

Conceptually the precompile is:

```solidity
// ExtendedOperations at address 0x64
function ValidateCiphertext(
    bytes1 metaData,
    uint256 ciphertextHigh,
    uint256 ciphertextLow,
    bytes calldata signature
) external returns (uint256 result);
```

**What the client does on COTI:** encrypt and sign, then submit. **What the chain does:** verify and either accept (`it*` → `gt*`) or revert.

Native COTI SDKs can encrypt locally with the account AES key. PoD dApps often call an HTTP encryption helper instead; that helper receives plaintext (see [Decryption trust model](decryption-trust-model.md)).

| | COTI | Zama / FHE (pinned stack) |
| :- | :--- | :------------------------ |
| Who verifies well-formed encryption | On-chain `validateCiphertext` | Coprocessors verify ZKPoK; host chain verifies coprocessor signatures |
| Client work | Encrypt + sign | Encrypt + prove (WASM), then Relayer round-trip |
| Host-chain encrypted input | ~192-byte `itUint256` | 32-byte handle + **230-byte** attestation (1 input, 3 signers) |
| Off-chain proof blob | None | **~18.8 KB** to Relayer |
| Failure before the host tx | No Relayer gate | Relayer / coprocessor / Gateway can reject the ZKPoK first |
| Failure in the host tx | Precompile revert | `InputVerifier` revert (bad signatures, threshold, binding) |

Related: [Decryption trust model](decryption-trust-model.md), [Transaction economics](transaction-economics.md), [Precompiles](../how-coti-works/advanced-topics/precompiles.md).
