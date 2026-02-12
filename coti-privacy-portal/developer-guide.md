# 🛠️ Developer Tutorial: Creating Private ERC20 Tokens on COTI

This guide explains how to create your own Privacy-Preserving Token on COTI L2 using the [`PrivateERC20`](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol) standard and how to correctly implement the `transferAndCall` pattern for secure transfers.

---

## 🚀 1. Creating a Private Token

To create a new private token, your contract needs to inherit from [`PrivateERC20`](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol). This base contract handles all the complex MPC (Multi-Party Computation) logic for encrypted balances and transfers.

### Basic Template

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "./PrivateERC20.sol";

contract MyPrivateToken is PrivateERC20 {
    
    // Define Roles (e.g., for a Bridge contract or Presale)
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() PrivateERC20("My Private Token", "pMYT") {
        // Grant the deployer the default admin role
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        
        // Grant the deployer the minter role
        _grantRole(MINTER_ROLE, msg.sender);
    }
}
```

### Key Features
*   **Inheritance**: `is [`PrivateERC20`](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol)` brings in all encrypted functionality (transfer, approve, burn).
*   **256-bit Support**: The standard supports full `uint256` encrypted values, allowing 1:1 mappings with public ERC20 tokens (18 decimals).
*   **Access Control**: Includes `AccessControl` to manage roles for minting/burning (e.g., for Bridge integrations).

---

## 🔒 2. Understanding Encrypted Operations

The [`PrivateERC20`](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol) contract relies on `MpcCore` to perform operations on encrypted data.

### Data Types
*   **`ctUint256`**: Ciphertext (Encrypted) Uint256. This is stored on-chain in `_balances`. User balances are always encrypted.
*   **`gtUint256`**: Garbled Text (Processable) Uint256. Used momentarily during MPC operations.
*   **`itUint256`**: Input Text. Data sent from a user (encryption + signature) to the contract.

### Example: Minting (Public -> Private)
Minting takes a public amount (e.g., from a Bridge or Initial Distribution) and converts it to encrypted tokens.

```solidity
function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
    // 1. Convert public uint256 to MPC-compatible type (gtUint256)
    gtUint256 memory gtAmount = MpcCore.setPublic256(amount);
    
    // 2. Internal mint function updates the encrypted balance using MPC addition
    _mint(to, gtAmount);
}
```

---

## 🔄 3. The `transferAndCall` Pattern

Standard ERC20 tokens often use the `approve` + `transferFrom` pattern. However, for privacy networks like COTI, this approach has usability and security drawbacks due to signature requirements.


### The Atomic Push Solution (`transferAndCall`)
Instead of `approve` (Pull), we use `transferAndCall` (Push).

1.  **User** calls `privateToken.transferAndCall(recipient, amount, data)`.
2.  **PrivateToken** transfers the tokens to the Recipient.
3.  **PrivateToken** calls `onTokenReceived` on the Recipient contract.
4.  **Recipient** executes its logic (e.g., Bridge processing, Swap, Staking) in the same transaction.

### Implementing `ITokenReceiver`

If you are building a contract that accepts Private Tokens, check out `ITokenReceiver`:

```solidity
import "./IPrivateERC20.sol"; // Contains ITokenReceiver

contract MyDeFiProtocol is ITokenReceiver {
    
    // Mapping to whitelist supported tokens
    mapping(address => bool) public supportedTokens;

    function onTokenReceived(
        address from, 
        uint256 amount, 
        bytes calldata data
    ) external override returns (bool) {
        // 1. Verify Sender (Security: Only accept whitelisted tokens)
        require(supportedTokens[msg.sender], "Unsupported token");
        
        // 2. Decode Data (if any instructions were sent)
        // (address intendedRecipient, uint256 minOut) = abi.decode(data, (address, uint256));
        
        // 3. Perform Logic
        // The tokens are ALREADY in this contract's balance (transferred in step 2).
        // e.g. Stake them, Swap them, etc.
        
        emit DepositReceived(from, msg.sender, amount);
        return true;
    }
}
```

### Frontend Integration (Client-Side)

When building a dApp, use the standard `ethers.js` signer.

```javascript
// JavaScript / TypeScript Example

// 1. Prepare Contract
const privateTokenContract = new ethers.Contract(tokenAddress, PRIVATE_TOKEN_ABI, signer);

// 2. Prepare Data
const amountWei = ethers.parseUnits("100", 18); // 100 Tokens
const data = "0x"; // Empty data (or encoded instructions)

// 3. Send Transaction
// Note: We set a high gas limit because MPC operations are computationally expensive.
const tx = await privateTokenContract.transferAndCall(
    recipientAddress, 
    amountWei, 
    data,
    { gasLimit: 5000000 } 
);

await tx.wait();
console.log("Transfer and Call successful!");
```

### Summary
*   **Use `transferAndCall`** for all contract interactions (Deposits, Swaps, Liquidity).
*   **Gas**: Always set a higher gas limit for MPC interactions (5M+ recommended for testnet).

## See Also

*   **[COTI Privacy Portal User Guide](user-guide.md)**: For a non-technical guide on bridging and managing tokens.
