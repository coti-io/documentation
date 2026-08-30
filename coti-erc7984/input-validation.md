# Input validation

Confidential transfers need a way to prove that an encrypted input is well-formed and bound to the caller. Zama’s FHE stack does that with a **client-generated zero-knowledge input proof**. COTI does it with an **on-chain precompile** that validates the ciphertext and signature — invalid inputs revert; the wallet never runs a heavy ZK prover.

## Zama: ZK input proof in the browser

Encryption and proof generation run in-client (WASM, often via a Web Worker). Multi-thread WASM needs `SharedArrayBuffer` and COOP/COEP headers; otherwise the SDK falls back to slower single-thread proving — painful on mobile browsers.

Illustrative TypeScript ([Zama SDK encrypt guide](https://docs.zama.org/protocol/sdk/guides/encrypt-decrypt)):

```typescript
import { ZamaSDK } from "@zama-fhe/sdk";

// Client must generate a ZK proof that the encryption is valid (ZKPoK).
// This is the expensive step on mobile / constrained browsers.
const { encryptedValues, inputProof } = await sdk.encrypt({
  values: [{ value: amount, type: "euint64" }],
  contractAddress,
  userAddress,
});

// Both ciphertext handle and inputProof are passed into the contract.
await token.confidentialTransfer(to, encryptedValues[0], inputProof);
```

Legacy pattern still common in examples:

```typescript
const enc = await fhevm
  .createEncryptedInput(contractAddress, userAddress)
  .add64(amount)
  .encrypt();
// enc.handles[0], enc.inputProof — proof generated on the device before submit
```

Proof payloads are large (often **tens of kilobytes** per input), which also hurts transaction economics. See [Transaction economics](transaction-economics.md).

## COTI: `validateCiphertext` on-chain

The user encrypts with their AES key and signs the input (`it*`: ciphertext + signature). The contract (or PoD path into COTI) calls the MPC precompile. If the ciphertext or signature is invalid, the call **reverts**. There is no client ZK proof of encryption validity.

Precompile address: `0x0000000000000000000000000000000000000064`.

Solidity (via `MpcCore`):

```solidity
import {MpcCore, itUint256, gtUint256} from "@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol";

function transfer(address to, itUint256 calldata value) external {
    // On-chain validation: bad ciphertext / signature → revert.
    // No ZK input proof was generated in the browser.
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

| | COTI | Zama / FHE |
| :- | :--- | :--------- |
| Who proves input validity | On-chain `validateCiphertext` | Client ZK `inputProof` |
| Client work | Encrypt + sign | Encrypt + prove (WASM) |
| Mobile / browser cost | Low | High (esp. without SharedArrayBuffer) |
| Failure mode | Transaction reverts | Proof never builds, or verify fails |
| Typical input size | ~192 bytes | Multi-KB with proof |

Related: [Decryption trust model](decryption-trust-model.md), [Precompiles](../how-coti-works/advanced-topics/precompiles.md).
