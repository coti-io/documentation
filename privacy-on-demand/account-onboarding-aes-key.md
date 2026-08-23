# Account onboarding (AES key)

Every PoD user needs an **account AES key** to decrypt `ct*` values returned from private operations. The key is a 16-byte AES secret represented as **32 hexadecimal characters without a `0x` prefix**.

This page describes a practical onboarding flow. Low-level crypto helpers live in [`@coti-io/coti-sdk-typescript`](coti-typescript-sdk-for-pod.md); decryption in app code typically goes through [`CotiPodCrypto`](typescript-pod-sdk.md).

## Trust model

- AES key recovery happens on a **trusted client** (browser, mobile app, or user-controlled wallet extension).
- A backend may broker encrypted key shares but must **never** see the plaintext AES key.
- Do not pass the recovered key in URLs, query parameters, or server logs.

## Typical flow

1. Client generates an RSA key pair.
2. Client sends the RSA **public** key to your onboarding service.
3. Service returns two encrypted key shares.
4. Client decrypts and combines shares into the account AES key.
5. Client stores the key in secure local storage.

## Example implementation

```typescript
import { generateRSAKeyPair, recoverUserKey } from "@coti-io/coti-sdk-typescript";

type KeySharesResponse = {
  encryptedKeyShare0: string;
  encryptedKeyShare1: string;
};

async function onboardAccount(
  requestShares: (publicKey: Uint8Array) => Promise<KeySharesResponse>
): Promise<string> {
  const { publicKey, privateKey } = generateRSAKeyPair();
  const { encryptedKeyShare0, encryptedKeyShare1 } = await requestShares(publicKey);

  const accountAesKey = recoverUserKey(privateKey, encryptedKeyShare0, encryptedKeyShare1);

  if (!/^[0-9a-fA-F]{32}$/.test(accountAesKey)) {
    throw new Error("invalid AES key format");
  }

  return accountAesKey;
}
```

## Storage recommendations

| Environment | Recommendation |
| --- | --- |
| Web dApp | Encrypted-at-rest in browser storage with strict origin and session controls |
| Mobile | OS keychain or secure enclave |
| Server-rendered apps | Avoid persisting the AES key on the server |

## Operational guardrails

- Never log key shares or the recovered key.
- Enforce TLS and authenticated onboarding endpoints.
- Define account recovery and key-rotation policy before production launch.
- Pair onboarding with [async UX guidance](async-private-operations.md): users cannot decrypt results until onboarding completes.

## See also

- [COTI TypeScript SDK for PoD](coti-typescript-sdk-for-pod.md) — `generateRSAKeyPair`, `recoverUserKey`, decrypt helpers
- [TypeScript PoD SDK](typescript-pod-sdk.md) — `CotiPodCrypto.decrypt`
- [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) — end-to-end walkthrough that assumes a key is available
