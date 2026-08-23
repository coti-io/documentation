# Reference: data types (`it*`, `ct*`, `gt*`)

Canonical Solidity source: [`MpcCore.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/utils/mpc/MpcCore.sol) in **`@coti-io/coti-contracts`**.

PoD uses three type families. Mixing them at the wrong boundary breaks signature validation, callback decode, or privacy guarantees. For plain-language definitions, see the [Glossary](glossary.md).

## Why the split exists

| Family | Role |
| --- | --- |
| **`it*`** | User-submitted encrypted input plus signature material |
| **`ct*`** | Encrypted value stored on EVM and decrypted client-side with the account AES key |
| **`gt*`** | Private compute-domain value used on COTI |

## `it*` (input payload)

Valid on EVM entrypoints and in `MpcAbiCodec` payloads.

| Type | Solidity shape | Typical use |
| --- | --- | --- |
| `itBool` | `{ ctBool ciphertext; bytes signature; }` | Signed encrypted bool input |
| `itUint8` … `itUint64` | `{ ctUint* ciphertext; bytes signature; }` | Signed encrypted scalar input |
| `itUint128` | `{ ctUint128 ciphertext; bytes[2] signature; }` | Signed 128-bit input |
| `itUint256` | `{ ctUint256 ciphertext; bytes[2][2] signature; }` | Signed 256-bit input |
| `itString` | `{ ctString ciphertext; bytes[] signature; }` | Signed encrypted string |

## `ct*` (ciphertext output)

Valid in EVM storage, callback payloads, and read APIs.

| Type | Solidity shape | Off-chain decrypt shape |
| --- | --- | --- |
| `ctBool` … `ctUint128` | `type ctUint* is uint256` | single `uint256` word (`bigint` or `0x` hex) |
| `ctUint256` | `struct { ctUint128 ciphertextHigh; ctUint128 ciphertextLow; }` | `{ ciphertextHigh, ciphertextLow }` tuple |
| `ctString` | `struct { ctUint64[] value; }` | array of 64-bit cells |

> **Note:** `ctUint128` is a user-defined value type wrapping one `uint256` word — not a struct of two limbs. Only `ctUint256` uses a two-limb struct.

## `gt*` (private compute domain)

Valid only on COTI-side contracts and executor operations. Never expose `gt*` in EVM public interfaces.

| Type | Solidity shape |
| --- | --- |
| `gtBool` … `gtUint128` | `type gtUint* is uint256` |
| `gtUint256` | `struct { gtUint128 high; gtUint128 low; }` |
| `gtString` | `struct { gtUint64[] value; }` |

## Conversion operations (`MpcCore`)

- `onBoard(ct*) → gt*` — move ciphertext into compute domain
- `offBoard(gt*) → ct*` — contract-held ciphertext
- `offBoardToUser(gt*, address) → ct*` — user-targeted decryptable ciphertext

## End-to-end boundary flow

1. Client builds `it*`.
2. EVM accepts `it*` and sends a request through the Inbox.
3. `MpcAbiCodec.reEncodeWithGt(...)` validates and converts `it*` to `gt*` argument shape.
4. COTI logic computes on `gt*`.
5. COTI returns `ct*`.
6. EVM callback stores `ct*`.
7. Client decrypts `ct*` with the account AES key.

## Critical gotcha: EVM `it*` maps to COTI `gt*`

For custom COTI-side logic:

- EVM function accepts `itUint64 amount`
- COTI function must be declared `gtUint64 amount`

Declaring COTI parameters as `it*` for methods invoked through the Inbox pipeline will not match ABI expectations.

## Required usage rules

1. EVM private inputs: `it*`
2. EVM private results: `ct*`
3. COTI private compute: `gt*`
4. Keep width consistent (`itUint64 → gtUint64 → ctUint64`)
5. Do not store plaintext sensitive values on EVM

## Frequent mistakes

- Wrong decrypt type or AES key for the stored `ct*` width
- Mismatch between frontend `DataType` and Solidity type width
- Decoding callback tuple as the wrong `ct*` type
- Skipping `requestId` correlation in callback handlers

## See also

- [TypeScript PoD SDK](typescript-pod-sdk.md) — `DataType` enum and `CotiPodCrypto`
- [Reference: PodLib primitives](reference-podlib-and-primitives.md)
- [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md)
