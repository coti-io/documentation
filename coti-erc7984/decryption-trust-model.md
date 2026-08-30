# Decryption trust model

Who can see plaintext when a user “decrypts” a confidential balance or result?

On **COTI**, user-facing ciphertexts (`ct*`) are bound to the account AES key. Decryption happens **on the client**. No single server in the path holds the user’s decrypted value.

On **Zama today**, user decrypt goes through a **Relayer over HTTPS** to the Gateway / threshold KMS. The KMS decrypts under the FHE network key and **re-encrypts** to the user’s transport public key; the app then recovers plaintext locally. The Relayer is documented as untrusted for plaintext, but a **networked decrypt/re-encrypt service still sits in the path**. That is an implementation architecture choice, not necessarily a permanent protocol law — document it as how the stack works today.

## COTI: local AES decrypt

Flow in short:

1. Onboard → account AES key (e.g. `GetUserKey` / wallet plugin).
2. Contract returns or stores `ct*` for that user (`offBoardToUser`).
3. Client reads the ciphertext from chain and decrypts locally (AES + XOR; see [AES keys](../how-coti-works/advanced-topics/aes-keys.md)).

```typescript
import { CotiPodCrypto, DataType } from "@coti/pod-sdk";

// Ciphertext already on-chain (e.g. from balanceOf / offBoardToUser).
const ciphertextHex = await token.balanceOf(userAddress); // ct* as returned by the contract

// Decrypt happens entirely in-process with the user's AES key.
// No HTTPS decrypt service is required to obtain plaintext.
const plain = CotiPodCrypto.decrypt(
  ciphertextHex.toString(),
  accountAesKey, // from onboarding — never leave the client
  DataType.Uint256
);

console.log("balance", plain);
```

**Implementation note:** encryption of inputs may still use a PoD encryption helper over HTTP to produce `it*` + signature; **decrypt of user `ct*` does not** send ciphertext to a server for plaintext recovery.

## Zama: Relayer HTTPS → KMS re-encrypt → client

Typical TypeScript shape with the current Zama SDK ([encrypt & decrypt guide](https://docs.zama.org/protocol/sdk/guides/encrypt-decrypt)):

```typescript
import { ZamaSDK } from "@zama-fhe/sdk";

// Encrypt (client WASM also builds a ZK input proof — see Input validation).
const { encryptedValues, inputProof } = await sdk.encrypt({
  values: [{ value: 1000n, type: "euint64" }],
  contractAddress,
  userAddress,
});

// User decrypt of a handle returned by the contract:
// This call goes Relayer HTTPS → Gateway / KMS.
// KMS decrypts under the FHE key and re-encrypts to the user's transport key;
// plaintext is recovered on the client only after that round trip.
const decrypted = await sdk.decryption.decryptValues([
  { encryptedValue: handleFromChain, contractAddress },
]);
```

Legacy Relayer SDK examples use `userDecrypt` / `createEncryptedInput(...).encrypt()` with the same architectural split: **proof + encrypt on the client**, **decrypt coordination over the Relayer**.

| | COTI | Zama (today) |
| :- | :--- | :----------- |
| Where user plaintext is recovered | Client, from on-chain `ct*` | Client, after Relayer/KMS re-encrypt hop |
| Server sees user plaintext? | No — AES key stays with the user | Relayer should not; KMS operates on FHE key material then re-encrypts |
| Dependency for decrypt | Chain read + local AES | HTTPS Relayer (API key / proxy on mainnet) + KMS |
| Protocol vs implementation | Local decrypt is the product model | Path can evolve; today’s apps depend on the Relayer |

For input proving cost (separate from decrypt), see [Input validation](input-validation.md).
