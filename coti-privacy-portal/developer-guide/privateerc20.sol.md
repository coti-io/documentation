# PrivateERC20.sol

### Overview

[`PrivateERC20`](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol) is an abstract Solidity contract that implements a fully encrypted ERC20 token on the COTI network.

Unlike a standard ERC20, where balances and transfer amounts are publicly visible on-chain, `PrivateERC20` stores balances and allowances as ciphertexts (encrypted values).

These values:

* can only be decrypted by the token holder
* use a personal AES key managed via the COTI Snap

All token operations, such as transfers, minting, and burning, are performed directly on encrypted data.

### Encrypted Computation

Encryption and arithmetic on encrypted values are handled by COTI's [`MpcCore`](../../how-coti-works/advanced-topics/precompiles.md) precompile at address `0x64`.

This precompile enables the network to perform operations such as:

* addition
* subtraction
* comparisons

on encrypted numbers without revealing the underlying plaintext, including to validators.

### How PrivateERC20 differs from ERC20

PrivateERC20 follows the ERC20 interface, but changes how data is stored and exposed.

| Feature                            | Standard ERC20 | PrivateERC20                            |
| ---------------------------------- | -------------- | --------------------------------------- |
| Balances visible on-chain          | Yes (public)   | No (encrypted ciphertext)               |
| Transfer amounts visible           | Yes            | No                                      |
| Allowances visible                 | Yes            | No                                      |
| `totalSupply()` returns real value | Yes            | No — always returns `0` for privacy     |
| Requires AES key to read balance   | No             | Yes                                     |
| Supports plain uint256 operations  | Yes            | Yes                                     |
| Supports encrypted operations      | No             | Yes (`itUint256`, `gtUint256` variants) |

For a comparison against the [ERC-7984](https://eips.ethereum.org/EIPS/eip-7984) confidential-token standard — and how COTI's Privacy on Demand `pERC20` compares with FHE-based implementations on encrypted-input size, numeric range, and allowance semantics — see [COTI ERC-7984](../../coti-erc7984/README.md).

### How it works

#### Balances

Balances are stored on-chain as `ctUint256` (ciphertext). Each account maintains two ciphertext values:

* `ciphertext` - used by the MPC network
* `userCiphertext` - re-encrypted for the user

#### Transfers

Transfers are executed via the MPC precompile using `MpcCore.transfer()`, which atomically updates both sender and receiver balances in encrypted form.

#### Reading your balance

To read a balance:

* call `balanceOf(address)` to retrieve the ciphertext
* decrypt it locally using your AES key via the COTI SDK

#### Writing encrypted amounts

Operations such as transfer, approve, mint, and burn require constructing an `itUint256`:

* an AES-encrypted value
* plus an ECDSA signature

This allows the MPC network to verify that the input was created by the rightful key holder.

#### Onboarding

Onboarding registers your wallet's encryption address on-chain via `setAccountEncryptionAddress()`.

This enables the contract to re-encrypt balances specifically for your AES key.
