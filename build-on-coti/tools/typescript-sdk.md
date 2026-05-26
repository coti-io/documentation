# TypeScript SDK

The [**COTI Typescript SDK**](https://github.com/coti-io/coti-sdk-typescript) provides a set of encryption, decryption, and cryptographic utilities, including RSA and AES encryption, message signing, and key handling functions. The utilities are primarily designed to work with cryptographic operations for secure communication and message signing, particularly within Ethereum smart contracts or similar environments.

## Features

* **AES encryption** with ECB mode for data of fixed block sizes.
* **RSA key pair generation**, encryption, and decryption using RSA-OAEP with SHA-256.
* **Signing of Ethereum transactions** using the `ethers` library's signing mechanisms.
* Utilities for encoding/decoding, padding, and cryptographic data manipulation.

## Clone, Build & Test

Full instructions for building and testing the package locally are available in the [coti-sdk-typescript GitHub repository](https://github.com/coti-io/coti-sdk-typescript)

## Installation

```bash
npm install @coti-io/coti-sdk-typescript
```

{% hint style="info" %}
SDK versions < 1.0.0 are for use with the Devnet, versions >= 1.0.0 are for use with the Testnet
{% endhint %}

## Usage

See the [**coti-typescript-examples**](https://github.com/coti-io/coti-typescript-examples) repository for code examples.

## Functions

```typescript
function encrypt(key: Uint8Array, plaintext: Uint8Array): { ciphertext: Uint8Array; r: Uint8Array }
```

Encrypts a given plaintext using the provided AES key. The plaintext is XORed with an encrypted random value.

* **Parameters:**
  * `key`: The AES encryption key (16 bytes).
  * `plaintext`: The data to be encrypted (must be 16 bytes or smaller).
* **Returns:** An object containing:
  * `ciphertext`: The encrypted data.
  * `r`: The random value used in the encryption process.

```typescript
function decrypt(key: Uint8Array, r: Uint8Array, ciphertext: Uint8Array): Uint8Array
```

Decrypts a ciphertext using the provided AES key and random value `r`.

* **Parameters:**
  * `key`: The AES encryption key (16 bytes).
  * `r`: The random value used during encryption (16 bytes).
  * `ciphertext`: The encrypted data (16 bytes).
* **Returns:** The decrypted plaintext.

<pre class="language-typescript"><code class="lang-typescript"><strong>function generateRSAKeyPair(): { publicKey: Uint8Array; privateKey: Uint8Array }
</strong></code></pre>

Generates a new RSA key pair (2048 bits) and returns the keys in DER format.

* **Returns:** An object containing:
  * `publicKey`: The RSA public key (DER-encoded).
  * `privateKey`: The RSA private key (DER-encoded).

```typescript
function decryptRSA(privateKey: Uint8Array, ciphertext: string): string
```

Decrypts an RSA-encrypted ciphertext using the provided private key.

* **Parameters:**
  * `privateKey`: The RSA private key (DER-encoded).
  * `ciphertext`: The encrypted ciphertext as a hex string.
* **Returns:** The decrypted message as a string.

```typescript
function sign(message: string, privateKey: string): Uint8Array
```

Signs a message using the provided Ethereum private key.

* **Parameters:**
  * `message`: The message to be signed.
  * `privateKey`: The Ethereum private key.
* **Returns:** A signature as a `Uint8Array` containing `r`, `s`, and `v` values.

```typescript
function signInputText(sender, contractAddress, functionSelector, ct: bigint): Uint8Array
```

Generates a signed message hash for Ethereum contract interactions.

* **Parameters:**
  * `sender`: The sender's information containing their wallet and user key.
  * `contractAddress`: The Ethereum contract address.
  * `functionSelector`: The function selector (bytes4) for the contract function.
  * `ct`: The ciphertext (big integer).
* **Returns:** A signature for the provided message.

```typescript
function buildInputText(plaintext: bigint, sender, contractAddress, functionSelector): itUint
```

Encrypts a plaintext (up to 64 bits) and generates a signed transaction payload.

* **Parameters:**
  * `plaintext`: The data to be encrypted (must be smaller than 64 bits).
  * `sender`: The sender's information containing their wallet and user key.
  * `contractAddress`: The Ethereum contract address.
  * `functionSelector`: The function selector for the contract function.
* **Returns:** An `itUint` object containing the encrypted ciphertext and signature.

```typescript
function buildStringInputText(plaintext: string, sender, contractAddress, functionSelector): itString
```

Encrypts a plaintext string and generates a signed transaction payload.

* **Parameters:**
  * `plaintext`: The data to be encrypted (string).
  * `sender`: The sender's information containing their wallet and user key.
  * `contractAddress`: The Ethereum contract address.
  * `functionSelector`: The function selector for the contract function.
* **Returns:** An `itString` object containing the encrypted ciphertext and signature.

```typescript
function decryptUint(ciphertext: ctUint, userKey: string): bigint
```

Decrypts an AES-encrypted ciphertext and returns the original plaintext as a `bigint`.

* **Parameters:**
  * `ciphertext`: The encrypted ciphertext.
  * `userKey`: The user key for AES decryption.
* **Returns:** The decrypted plaintext as a `bigint`.

```typescript
function decryptString(ciphertext: { value: bigint[] }, userKey: string): string
```

Decrypts an AES-encrypted ciphertext representing a string.

* **Parameters:**
  * `ciphertext`: An object containing the encrypted ciphertext as a list of bigints.
  * `userKey`: The user key for AES decryption.
* **Returns:** The decrypted plaintext as a string.

<pre class="language-typescript"><code class="lang-typescript"><strong>function generateRandomAesKeySizeNumber(): string
</strong></code></pre>

Generates a random 128-bit AES key.

* **Returns:** A string containing the random bytes.

## High-Level DApp Utilities

The SDK also includes high-level utilities commonly needed by front-end applications and wallet integrations.

```typescript
function normalizeAesKey(aesKey: string): string
```

Normalizes an AES key string by removing the `0x` prefix, lowering its case, and ensuring it holds valid hexadecimal characters representation (for exactly 128 bits or 256 bits).

* **Parameters:**
  * `aesKey`: The user's AES key as a hex string.
* **Returns:** The normalized AES key.
* **Throws:** An HTTP error or RangeError when the AES key is invalid.

```typescript
function decryptCtUint64(ciphertext: ctUint, aesKey: string, options?: DecryptionOptions): bigint | null
```

A robust wrapper around `decryptUint` that cleanly handles zero balances and applies a sanity check (`isInsaneDecryptedValue`) to prevent garbage output when a wrong AES key is accidentally supplied.

* **Parameters:**
  * `ciphertext`: The encrypted 64-bit ciphertext.
  * `aesKey`: The user key for decryption.
  * `options`: Optional arguments like `decimals` and `insaneThresholdBase` to tune the sanity boundary limits. 
* **Returns:** The decrypted big integer, or `null` if the decryption breached sanity threshold limits.

```typescript
function buildItSignature(signerAddress: string, contractAddress: string, functionSelector: string, ciphertext: bigint, privateKey: string): string
```

Builds the standardized COTI IT signature required by encrypted parameters in COTI L2 smart contracts, computing `solidityPackedKeccak256` and constructing the final properly padded EVM verification signature mapping 27/28 to 0/1.

* **Parameters:**
  * `signerAddress`: The signer's wallet public address.
  * `contractAddress`: Target COTI L2 smart-contract address.
  * `functionSelector`: The EVM 4-byte selector.
  * `ciphertext`: The resulting input-text encoded `bigint`.
  * `privateKey`: Signer's private key string.
* **Returns:** A properly normalized 65-byte hex string (including the v-byte mapping padding) ready to be embedded as the IT signature.
