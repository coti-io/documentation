# Private ERC20

The [**Private ERC20**](https://github.com/coti-io/coti-contracts/blob/main/contracts/token/PrivateERC20/PrivateERC20.sol) contract is an abstract implementation of a privacy-enhanced ERC20 token. It introduces mechanisms for handling encrypted balances, allowing for more secure and private token transfers. This contract integrates with the MPC Core library for secure multiparty computation (MPC).

## Usage

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.19;

import "@coti-io/coti-contracts/contracts/token/PrivateERC20/PrivateERC20.sol";

contract MyToken is PrivateERC20 {
    constructor() PrivateERC20("Private Token", "PTOK") {}
}
```

## Functions

```solidity
constructor(string memory name_, string memory symbol_)
```

```solidity
function name() view returns (string memory)
```

```solidity
function symbol() view returns (string memory)
```

```solidity
function decimals() view returns (uint8)
```

```solidity
function totalSupply() view returns (uint256)
```

```solidity
function accountEncryptionAddress(address account) view returns (address)
```

```solidity
function balanceOf(address account) view returns (ctUint256)
```

```solidity
function balanceOf() returns (gtUint256)
```

```solidity
function setAccountEncryptionAddress(address addr) returns (bool)
```

```solidity
function transfer(address to, itUint256 calldata value) returns (gtBool)
```

```solidity
function transfer(address to, gtUint256 value) returns (gtBool)
```

```solidity
function allowance(address owner, address spender) view returns (Allowance memory)
```

```solidity
function allowance(address account, bool isSpender) returns (gtUint256)
```

```solidity
function reencryptAllowance(address account, bool isSpender) returns (bool)
```

```solidity
function approve(address spender, itUint256 calldata value) returns (bool)
```

```solidity
function approve(address spender, gtUint256 value) returns (bool)
```

```solidity
function transferFrom(address from, address to, itUint256 calldata value) returns (gtBool)
```

```solidity
function transferFrom(address from, address to, gtUint256 value) returns (gtBool)
```

```solidity
function _transfer(address from, address to, gtUint256 value) returns (gtBool)
```

```solidity
function _update(address from, address to, gtUint256 value) returns (gtBool)
```

```solidity
function _getBalance(address account) returns (gtUint256)
```

```solidity
function _getAccountEncryptionAddress(address account) view returns (address)
```

```solidity
function _updateBalance(address account, gtUint256 balance)
```

```solidity
function _mint(address account, gtUint256 value) returns (gtBool)
```

```solidity
function _burn(address account, gtUint256 value) returns (gtBool)
```

```solidity
function _approve(address owner, address spender, gtUint256 value)
```

```solidity
function _spendAllowance(address owner, address spender, gtUint256 value)
```

```solidity
function _safeOnboard(ctUint256 value) returns (gtUint256)
```

```solidity
event Transfer(address indexed from, address indexed to, ctUint256 senderValue, ctUint256 receiverValue);
```

```solidity
event Approval(address indexed owner, address indexed spender, ctUint256 ownerValue, ctUint256 spenderValue);
```

## Errors

```solidity
error ERC20InvalidSender(address sender)
```

```solidity
error ERC20InvalidReceiver(address receiver)
```

```solidity
error ERC20InvalidApprover(address approver)
```

```solidity
error ERC20InvalidSpender(address spender)
```
