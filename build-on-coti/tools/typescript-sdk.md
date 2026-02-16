# TypeScript SDK

The [**COTI Typescript SDK**](https://github.com/coti-io/coti-sdk-typescript) is a cryptographic toolkit for building privacy-preserving applications on the COTI blockchain. It provides AES and RSA encryption, message signing, and key handling utilities—enabling developers to encrypt sensitive transaction inputs before submitting them to smart contracts and decrypt encrypted data retrieved from the blockchain.

## Features

*   **Privacy-Preserving Transactions**: Encrypt sensitive data (amounts, IDs) before submission, ensuring confidentiality on-chain.
*   **Smart Contract Ready**: Generates encrypted inputs (`itUint`, `itString`) formatted specifically for COTI's confidentiality contracts.
*   **Type Safety**: Full TypeScript support with comprehensive definitions for all encrypted structures (`it` and `ct` types).
*   **Broad Support**: Handle 128-bit/256-bit integers, booleans, and arbitrary strings.

## Data Types

The SDK uses specific structures for privacy-preserving inputs (`it*`) and on-chain ciphertext outputs (`ct*`):

*   **`it` (Input Text)**: Encrypted input data (ciphertext + signature) sent **to** the blockchain.
*   **`ct` (Ciphertext)**: Encrypted output data received **from** the blockchain.

| Input Type (`it`) | Result Type (`ct`) | Underlying Plaintext                             |
| :---------------- | :----------------- | :----------------------------------------------- |
| `itBool`          | `ctBool`           | Boolean (stored as BigInt: `0n` or `1n`)         |
| `itUint`          | `ctUint`           | Unsigned Integer (up to 128-bit)                 |
| `itUint256`       | `ctUint256`        | Unsigned Integer (up to 256-bit, split high/low) |
| `itString`        | `ctString`         | UTF-8 String (chunked into 8-byte segments)      |



## Installation

```bash
npm install @coti-io/coti-sdk-typescript
```

### Requirements

- Node.js >= 18.20.5
- TypeScript (for TypeScript projects)



## Usage

See the [coti-typescript-examples](https://github.com/coti-io/coti-typescript-examples) repository for code examples.

## Functions

### `prepareIT`

```typescript
function prepareIT(
  plaintext: bigint,
  sender: { wallet: BaseWallet; userKey: string },
  contractAddress: string,
  functionSelector: string
): itUint
```

Encrypts a 128-bit unsigned integer (or boolean) and generates a signed transaction payload.

*   **Parameters:**
    *   `plaintext`: The data to be encrypted (max 128 bits). For booleans, use `1n` (true) or `0n` (false).
    *   `sender`: Object containing the ethers.js `wallet` and the 32-byte hex `userKey`.
    *   `contractAddress`: The smart contract address.
    *   `functionSelector`: The 4-byte function selector (e.g., `0xa9059cbb`).
*   **Returns:** An `itUint` object containing `ciphertext` (BigInt) and `signature` (Uint8Array/string).

**Example:**
```typescript
const { ciphertext, signature } = prepareIT(12345n, { wallet, userKey }, contractAddress, funcSelector)
```

### `prepareIT256`

```typescript
function prepareIT256(
  plaintext: bigint,
  sender: { wallet: BaseWallet; userKey: string },
  contractAddress: string,
  functionSelector: string
): itUint256
```

Encrypts a 256-bit unsigned integer and generates a signed transaction payload.

*   **Parameters:** Same as `prepareIT`, but supports values up to 256 bits.
*   **Returns:** An `itUint256` object containing `ciphertext` (split into high/low BigInts) and `signature`.

**Example:**
```typescript
const { ciphertext, signature } = prepareIT256(2n**200n, { wallet, userKey }, contractAddress, funcSelector)
```

### `buildStringInputText`

```typescript
function buildStringInputText(
  plaintext: string,
  sender: { wallet: BaseWallet; userKey: string },
  contractAddress: string,
  functionSelector: string
): itString
```

Encrypts a string (UTF-8 supported) and generates a signed transaction payload.

*   **Parameters:**
    *   `plaintext`: The string to encrypt.
    *   `sender`, `contractAddress`, `functionSelector`: As above.
*   **Returns:** An `itString` object containing chunks of encrypted data and signatures.

**Example:**
```typescript
const { ciphertext, signature } = buildStringInputText("Hello COTI", { wallet, userKey }, contractAddress, funcSelector)
```

### `decryptUint`

```typescript
function decryptUint(ciphertext: ctUint, userKey: string): bigint
```

Decrypts a 128-bit ciphertext (or boolean).

*   **Parameters:**
    *   `ciphertext`: The encrypted BigInt returned from the contract.
    *   `userKey`: The 32-byte hex user key.
*   **Returns:** The decrypted BigInt.

**Example:**
```typescript
const val = decryptUint(encryptedVal, userKey)
```

### `decryptUint256`

```typescript
function decryptUint256(ciphertext: ctUint256, userKey: string): bigint
```

Decrypts a 256-bit ciphertext.

*   **Parameters:**
    *   `ciphertext`: Object containing `ciphertextHigh` and `ciphertextLow`.
    *   `userKey`: The 32-byte hex user key.
*   **Returns:** The decrypted BigInt.

### `decryptString`

```typescript
function decryptString(ciphertext: ctString, userKey: string): string
```

Decrypts an encrypted string.

*   **Parameters:**
    *   `ciphertext`: Object containing an array of BigInt values.
    *   `userKey`: The 32-byte hex user key.
*   **Returns:** The decrypted string.

### `encrypt`

```typescript
function encrypt(key: Uint8Array, plaintext: Uint8Array): { ciphertext: Uint8Array; r: Uint8Array }
```
Encrypts a 16-byte block using AES-ECB with a random value.

### `decrypt`

```typescript
function decrypt(key: Uint8Array, r: Uint8Array, ciphertext: Uint8Array, r2?: Uint8Array, ciphertext2?: Uint8Array): Uint8Array
```
Decrypts an AES-encrypted block (or two blocks for 256-bit support).

### `generateRSAKeyPair`

```typescript
function generateRSAKeyPair(): { publicKey: Uint8Array; privateKey: Uint8Array }
```
Generates a 2048-bit RSA key pair (DER-encoded).

### `decryptRSA`

```typescript
function decryptRSA(privateKey: Uint8Array, ciphertext: string): string
```
Decrypts an RSA-encrypted hex string.

### `sign`

```typescript
function sign(message: string, privateKey: string): Uint8Array
```
Signs a message with an Ethereum private key.

### `recoverUserKey`

```typescript
function recoverUserKey(privateKey: Uint8Array, encryptedKeyShare0: string, encryptedKeyShare1: string): string
```
Recovers the AES user key from encrypted shares.

### `buildInputText`

```typescript
function buildInputText(plaintext: bigint, sender, contractAddress, functionSelector): itUint
```
**Note:** Optimized specifically for **64-bit integers**. Use `prepareIT` for general 128-bit support.

---
