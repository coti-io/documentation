# Decryption trust model

Who can see plaintext, and which services sit on the path?

**Decrypt of user `ct*` on COTI is local:** the client reads ciphertext from chain and AES-decrypts with the account key. That step does not send ciphertext to a server for plaintext recovery.

**Encrypt is a separate path.** Native COTI SDKs can encrypt locally with the same AES key. PoD dApps that call `CotiPodCrypto.encrypt` **POST the plaintext** to the PoD encryption HTTP service (`buildEncryptedInputs`). That helper **does receive plaintext**. Do not treat “client-side decrypt” as “no server ever sees the value.”

Zama user decrypt goes Relayer HTTPS → Gateway / threshold KMS (re-encrypt to the user’s transport key) → client. The Relayer is documented as untrusted for plaintext; the KMS and coprocessors are the cryptographic trust base.

## Trust assumptions

| | COTI | Zama / FHEVM (pinned stack, 6 Sep 2026) |
| :- | :--- | :-------------------------------------- |
| Who holds the network key | MPC / garbled-circuit nodes hold **threshold shares** of the network AES key. No single node is documented as able to reconstruct it ([AES keys](../how-coti-works/advanced-topics/aes-keys.md)). | Threshold **coprocessors** attest inputs (`InputVerifier` signatures). Threshold **KMS** decrypts under the FHE network key. |
| User key | Account AES key from onboarding (`GetUserKey` / wallet plugin). The client must store it; loss means those `ct*` values cannot be decrypted. | User transport key for KMS re-encrypt. ACL on handles controls who may request decrypt. |
| Encrypt-time plaintext | **PoD:** encryption HTTP service reads the plaintext. **Native COTI:** `encryptValue` can stay on the client. | Client WASM encrypts under the FHE public key. Relayer sees packed ciphertext + ZKPoK, not the plaintext. |
| Decrypt-time plaintext | Client AES on `ct*`. No Relayer decrypt hop. | Client recovers plaintext after KMS re-encrypt. Relayer/Gateway are on the path (liveness). |
| Liveness | PoD also depends on Inbox / relayer for **request and callback**. Native COTI decrypt does not. | Relayer + Gateway for **input registration and decrypt**. Failure can happen before a host tx exists ([Input validation](input-validation.md)). |

## COTI: local AES decrypt

1. Onboard → account AES key.
2. Contract returns or stores `ct*` for that user (`offBoardToUser` / PoD callback).
3. Client reads the ciphertext from chain and decrypts locally (AES + XOR).

```typescript
import { CotiPodCrypto, DataType } from "@coti-io/pod-sdk";

// Ciphertext already on-chain (e.g. from balanceOf / offBoardToUser).
const ciphertextHex = await token.balanceOf(userAddress);

// Decrypt is in-process with the user's AES key.
const plain = CotiPodCrypto.decrypt(
  ciphertextHex.toString(),
  accountAesKey,
  DataType.Uint256
);
```

```typescript
// PoD encrypt: plaintext is in the HTTP body. The encryption service reads it.
const enc = await CotiPodCrypto.encrypt(
  "1000",
  "testnet",
  DataType.itUint256
);
```

## Zama: Relayer HTTPS → KMS re-encrypt → client

Typical TypeScript ([Zama encrypt & decrypt](https://docs.zama.org/protocol/sdk/guides/encrypt-decrypt)):

```typescript
import { ZamaSDK } from "@zama-fhe/sdk";

const { encryptedValues, inputProof } = await sdk.encrypt({
  values: [{ value: 1000n, type: "euint64" }],
  contractAddress,
  userAddress,
});

// User decrypt of a handle: Relayer HTTPS → Gateway / KMS re-encrypt.
const decrypted = await sdk.decryption.decryptValues([
  { encryptedValue: handleFromChain, contractAddress },
]);
```

**Public** decrypt (plaintext that a contract can `require` on) is a different flow: off-chain KMS signatures, then a **later** host transaction. See [On-chain data availability](on-chain-data-availability.md) and COTI native [synchronous decrypt](../how-coti-works/advanced-topics/coti-vs-others.md#advantages-over-fhe).
