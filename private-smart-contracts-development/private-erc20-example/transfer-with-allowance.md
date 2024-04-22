# 🪪 Transfer With Allowance

The function transfers the given amount from the given address to the receiver address, if the sender allow to do it.

There are four types of transfer:

```solidity
1. function transferFrom(address _from, address _to, ctUint64 _itCT, bytes calldata _itSignature, bool revealRes) public returns (bool success)
```

```solidity
2. function transferFrom(address _from, address _to, uint64 _value, bool revealRes) public returns (bool success)
```

```solidity
3. function contractTransferFrom(address _from, address _to, gtUint64 _value) public returns (gtBool success)
```

```solidity
4. function contractTransferFromClear(address _from, address _to, uint64 _value) public returns (gtBool success)
```

* 1: Validates the inputText to get the required amount to transfer.
* 3: Transfer an encrypted amount. This version is used by a contract that already has the value as encrypted gtUint64.

**Parameters:**&#x20;

* \_from: The sender address
* \_to: The receiver address
* \_itCT, \_itSignature: An encrypted value to transfer as an InputText&#x20;
* \_value: The value to transfer (clear or encrypted depends on the type)
* revealRes: A boolean indicates weather to reveal the result or not.

**Return value:**&#x20;

* 1,2: If revealRes is true, the result (success/failure) is decrypted and returned. Else, return true.
* 3,4: Return the gtBool returned by the transfer operation.

## **Full Solidity Code**

```solidity
1. 
// Transfers the amount of tokens given inside the IT (encrypted and signed value) 
// from address _from to address _to
// params: _from: the address to transfer from
//         __to: the address to transfer to
//         _itCT: the encrypted value of the amount to transfer
//         _itSignature: the signature of the amount to transfer
//         revealRes: indicates if we should reveal the result of the transfer
// returns: In case revealRes is true, returns the result of the transfer. 
// In case revealRes is false, always returns true
function transferFrom(address _from, address _to, ctUint64 _itCT, 
            bytes calldata _itSignature, bool revealRes) public returns (bool success){
    // Create IT from ciphertext and signature
    itUint64 memory it;
    it.ciphertext = _itCT;
    it.signature = _itSignature;
    // Verify the IT and transfer the value
    gtBool result = contractTransferFrom(_from, _to, MpcCore.validateCiphertext(it));
    if (revealRes){
        return MpcCore.decrypt(result);
    } else {
        return true;
    }
}
```

```solidity
2.
// Transfers the amount of tokens from address _from to address _to
// params: _from: the address to transfer from
//         __to: the address to transfer to
//         _value: the value of the amount to transfer
//         revealRes: indicates if we should reveal the result of the transfer
// returns: In case revealRes is true, returns the result of the transfer. In case revealRes is false, always returns true
function transferFrom(address _from, address _to, uint64 _value, bool revealRes) 
                      public returns (bool success){
    gtBool result = contractTransferFromClear(_from, _to, _value);
    if (revealRes){
        return MpcCore.decrypt(result);
    } else {
        return true;
    }
}
```

```solidity
3. 
// Transfers the amount of tokens given inside the encrypted value from address _from 
// to address _to
// params: _from: the address to transfer from
//         _to: the address to transfer to
//         _value: the encrypted value of the amount to transfer
// returns: The encrypted result of the transfer.
function contractTransferFrom(address _from, address _to, gtUint64 _value) 
                              public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(_from, _to);
    gtUint64 allowance = MpcCore.onBoard(getGTAllowance(_from, msg.sender));
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result, gtUint64 newAllowance) 
                = MpcCore.transferWithAllowance(fromBalance, toBalance, _value, allowance);

    setApproveValue(_from, msg.sender, MpcCore.offBoard(newAllowance));
    emit Transfer(_from, _to);
    setNewBalances(_from, _to, newFromBalance, newToBalance);

    return result;
}
```

```solidity
4.
// Transfers the amount of tokens from address _from to address _to
// params: _from: the address to transfer from
//         _to: the address to transfer to
//         _value: the value of the amount to transfer
// returns: The encrypted result of the transfer.
function contractTransferFromClear(address _from, address _to, uint64 _value) 
                                   public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(_from, _to);
    gtUint64 allowance = MpcCore.onBoard(getGTAllowance(_from, msg.sender));
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result, gtUint64 newAllowance) 
            = MpcCore.transferWithAllowance(fromBalance, toBalance, _value, allowance);

    setApproveValue(_from, msg.sender, MpcCore.offBoard(newAllowance));
    emit Transfer(_from, _to, _value);
    setNewBalances(_from, _to, newFromBalance, newToBalance);

    return result;
}
```

## **Example Usage:**

{% tabs %}
{% tab title="Python" %}
```python
print("************* Transfer clear ", plaintext_integer, " from my account to Alice *************")
# Transfer 5 SOD to Alice
function = contract.functions.transferFrom(account.address, alice_address.address, plaintext_integer, True)
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'transfer from', INITIAL_BALANCE - 4*plaintext_integer)

print("************* Transfer clear ", plaintext_integer, " from my account to Alice *************")
# Transfer 5 SOD to Alice
function = contract.functions.contractTransferFromClear(account.address, alice_address.address, plaintext_integer)
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'contract transfer from clear ', INITIAL_BALANCE - 5*plaintext_integer)

print("************* Transfer IT ", plaintext_integer, " from my account to Alice *************")
# Transfer 5 SOD to Alice
function = contract.functions.transferFrom(account.address, alice_address.address, dummyCT, dummySignature, False) # Dummy function
func_sig = get_function_signature(function.abi) # Get the function signature
# Prepare the input text for the function
ct, signature = prepare_IT(plaintext_integer, user_key, account, contract, func_sig, bytes.fromhex(private_key[2:]))
# Create the real function using the prepared IT
function = contract.functions.transferFrom(account.address, alice_address.address, ct, signature, False)
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'transfer from IT', INITIAL_BALANCE - 6*plaintext_integer)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
console.log("************* Transfer clear ", plaintext_integer, " from my account to Alice *************")
// Transfer 5 SOD to Alice
func = contract.methods.transferFrom(account.address, alice_address.address, plaintext_integer, true)
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - 4*plaintext_integer)

console.log("************* Transfer clear ", plaintext_integer, " from my account to Alice *************")
// Transfer 5 SOD to Alice
func = contract.methods.contractTransferFromClear(account.address, alice_address.address, plaintext_integer)
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - 5*plaintext_integer)

console.log("************* Transfer IT ", plaintext_integer, " from my account to Alice *************");
// Transfer 5 SOD to Alice
func = contract.methods.transferFrom(account.address, alice_address.address, dummyCT, dummySignature, false);
hashFuncSig = getFunctionSignature(func);
({ctInt, signature} = prepareIT(plaintext_integer, user_key, account.address, contract.options.address, hashFuncSig, Buffer.from(SIGNING_KEY.slice(2), 'hex')));
// Create the real function using the prepared IT output
func = contract.methods.transferFrom(account.address, alice_address.address, ctInt, signature, false);
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - 6*plaintext_integer);
```
{% endtab %}
{% endtabs %}
