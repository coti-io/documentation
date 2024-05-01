# Smart Contracts

## What is a Smart Contract?

When deployed to a blockchain, a _smart contract_ is a set of instructions that can be executed without intervention from third parties. The smart contract code defines how it responds to input, just like the code of any other computer program.

A valuable feature of smart contracts is that they can store and manage onchain assets (like [ETH or ERC20 tokens](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)), just like you can with an Ethereum wallet. Because they have an onchain address like a wallet, they can do everything any other address can. This enables you to program automated actions when receiving and transferring assets.

Smart contracts can connect to real-world market prices of assets to produce powerful applications. Securely connecting smart contracts with offchain data and services is what makes them _hybrid_ smart contracts.

{% embed url="https://youtu.be/ZE2HxTmxfrI?si=T5DmLId6u1k6oqRG" %}

## The language of smart contracts

The most popular language for writing smart contracts on Ethereum and EVM Chains like COTI is [Solidity](https://docs.soliditylang.org/en/v0.8.7/). It was created by the Ethereum Foundation specifically for smart contract development and is constantly being updated. Other languages exist for writing smart contracts on Ethereum and EVM Chains, but Solidity is the language used for COTI smart contracts.

If you've ever written Javascript, Java, or other object-oriented scripting languages, Solidity should be easy to understand. Similar to object-oriented languages, Solidity is considered to be a _contract_-oriented language.

## What does a smart contract look like?

The structure of a smart contract is similar to that of a class in Javascript, with a few differences. For example, the following `HelloWorld` contract is a simple smart contract that stores a single variable and includes a function to update the value of that variable.

```
// SPDX-License-Identifier: MIT

pragma solidity 0.8.19;

/**
 * THIS IS AN EXAMPLE CONTRACT THAT USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

contract HelloWorld {
    string public message;

    constructor(string memory initialMessage) {
        message = initialMessage;
    }

    function updateMessage(string memory newMessage) public {
        message = newMessage;
    }
}

```

## Solidity versions

The first thing that every Solidity file must have is the Solidity version definition. The `HelloWorld.sol` contract uses version `0.8.19`, which is defined in the contract as `pragma solidity 0.8.19;`

You can see the latest versions of the Solidity compiler [here](https://github.com/ethereum/solc-bin/blob/gh-pages/bin/list.txt/?target=\_blank). You might also notice smart contracts that are compatible with a range of versions.

```solidity
pragma solidity >=0.7.0 <0.9.0;
```

This means that the code is written for Solidity version 0.7.0, or a newer version of the language up to, but not including version 0.9.0. The `pragma` selects the compiler, which defines how the code is treated.

## Naming a contract

The `contract` keyword defines the name of the contract, which in this example is `HelloWorld`. This is similar to declaring a `class` in Javascript. The implementation of `HelloWorld` is inside this definition and denoted with curly braces.

```solidity
contract HelloWorld {

}
```

## Variables

Like Javascript, contracts can have state variables and local variables. **State variables** are variables with values that are permanently stored in contract storage. The values of **local variables**, however, are present only until the function is executing. There are also different types of variables you can use within Solidity, such as `string`, `uint256`, etc. Check out the [Solidity documentation](https://docs.soliditylang.org/en/v0.8.7/) to learn more about the different kinds of variables and types.

_Visibility modifiers_ are used to define the level of access to these variables. Here are some examples of state variables with different visibility modifiers:

```solidity
string public message;
uint256 internal internalVar;
uint8 private privateVar;
```

Learn more about state variables visibility [here](https://docs.soliditylang.org/en/latest/contracts.html#state-variable-visibility).

## Constructors

Another familiar concept to programmers is the **constructor**. When you deploy a contract, the constructor sets the state of the contract when it is first created.

In `HelloWorld`, the constructor takes in a `string` as a parameter and sets the `message` state variable to that string.

```solidity
constructor(string memory initialMessage) {
  message = initialMessage;
}
```

## Functions

**Functions** can access and modify the state of the contract or call other functions on external contracts. `HelloWorld` has a function named `updateMessage`, which updates the current message stored in the state.

```solidity
constructor(string memory initialMessage) {
  message = initialMessage;
}

function updateMessage(string memory newMessage) public {
  message = newMessage;
}
```

Functions use visibility modifiers to define the access level. Learn more about functions visibility [here](https://docs.soliditylang.org/en/latest/contracts.html#function-visibility).

## Interfaces

An **interface** is another concept that is familiar to programmers of other languages. Interfaces define functions without their implementation, which leaves inheriting contracts to define the actual implementation themselves. This makes it easier to know what functions to call in a contract. Here's an example of an interface:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

/**
 * THIS IS AN EXAMPLE CONTRACT THAT USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

interface numberComparison {
    function isSameNum(uint a, uint b) external view returns (bool);
}

contract Test is numberComparison {
    constructor() {}

    function isSameNum(uint a, uint b) external pure override returns (bool) {
        if (a == b) {
            return true;
        } else {
            return false;
        }
    }
}
```

For this example, `override` is necessary in the `Test` contract function because it overrides the base function contained in the `numberComparison` interface. The contract uses `pure` instead of `view` because the `isSameNum` function in the `Test` contract does not return a storage variable.

## What does "deploying" mean?

**Deploying** a smart contract is the process of pushing the code to the blockchain, at which point it resides with an onchain address. Once it's deployed, the code cannot be changed and is said to be _immutable_.
