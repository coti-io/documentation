# COTI TypeScript SDK for PoD

Package: **`@coti-io/coti-sdk-typescript`**

This is the **low-level cryptography layer** beneath [`@coti/pod-sdk`](typescript-pod-sdk.md). Use it when you need signed `it*` payload construction, account key recovery, or direct decrypt helpers. Most PoD apps call `CotiPodCrypto` instead, which delegates decrypt to this package.

```bash
npm install @coti-io/coti-sdk-typescript
```

## APIs commonly used with PoD

| API | Purpose |
| --- | --- |
| `buildInputText` | Build signed encrypted numeric input payload |
| `buildStringInputText` | Build signed encrypted string input payload |
| `decryptUint` / `decryptUint256` | Decrypt numeric ciphertext with account AES key |
| `decryptString` | Decrypt string ciphertext with account AES key |
| `generateRSAKeyPair` | Generate onboarding RSA keys |
| `recoverUserKey` | Reconstruct account AES key from encrypted shares |

See [Account onboarding (AES key)](account-onboarding-aes-key.md) for the recovery flow.

## Safer selector handling

Avoid hardcoding function selectors. Build from the contract ABI:

```typescript
import { Interface, Wallet } from "ethers";
import { buildInputText } from "@coti-io/coti-sdk-typescript";

const iface = new Interface(["function compare((uint256,bytes),(uint256,bytes))"]);
const selector = iface.getFunction("compare")!.selector;

const sender = {
  wallet: new Wallet(process.env.PRIVATE_KEY!),
  userKey: accountAesKey,
};

const itValue = buildInputText(42n, sender, contractAddress, selector);
```

The `contractAddress` and `selector` used when signing must match the on-chain call target exactly.

## String input example

```typescript
import { buildStringInputText } from "@coti-io/coti-sdk-typescript";

const itMemo = buildStringInputText("private memo", sender, contractAddress, selector);
```

## Decryption example

```typescript
import { decryptUint, decryptString } from "@coti-io/coti-sdk-typescript";

const amount = decryptUint(BigInt(ctAmountHex), accountAesKey);
const note = decryptString({ value: ctStringCells }, accountAesKey);
```

For `ctUint256` limbs, prefer `decryptUint256({ ciphertextHigh, ciphertextLow }, accountAesKey)` (see [Reference: data types](reference-data-types.md)).

## Relationship to `@coti/pod-sdk`

| Task | Package |
| --- | --- |
| Encrypt plaintext via PoD encryption HTTP service | `@coti/pod-sdk` (`CotiPodCrypto.encrypt`) |
| Decrypt `ct*` in app code | `@coti/pod-sdk` (`CotiPodCrypto.decrypt`) or this package directly |
| Onboard user / recover AES key | This package |
| Fee estimate, submit tx, track request | `@coti/pod-sdk` (`PodContract`, `PodRequest`) |

## See also

- [Account onboarding (AES key)](account-onboarding-aes-key.md)
- [TypeScript PoD SDK](typescript-pod-sdk.md)
