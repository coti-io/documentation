# Transfer

The function transfers the given amount to the receiver address.

There are four types of transfer:

<pre class="language-solidity"><code class="lang-solidity"><strong>1. function transfer(address _to, ctUint64 _itCT, bytes calldata _itSignature, bool revealRes) public returns (bool success);
</strong></code></pre>

```solidity
2. function transfer(address _to, uint64 _value, bool revealRes) public returns (bool success);
```

```solidity
3. function contractTransfer(address _to, gtUint64 _value) public returns (gtBool success)
```

```solidity
4. function contractTransferClear(address _to, uint64 _value) public returns (gtBool success)
```

* 1: Validates the inputText to get the required amount to transfer.
* 3: Transfer an encrypted amount. This version is used by a contract that already has the value as encrypted gtUint64.

**Parameters:**

* \_to: The receiver address
* \_itCT, \_itSignature: An encrypted value to transfer as an InputText
* \_value: The value to transfer (clear or encrypted depends on the type)
* revealRes: A boolean indicates weather to reveal the result or not.

**Return value:**

* 1,2: If revealRes is true, the result (success/failure) is decrypted and returned. Else, return true.
* 3,4: Return the gtBool returned by the transfer operation.

## Full Solidity Code

<pre class="language-solidity"><code class="lang-solidity">1.
// Transfers the amount of tokens given inside the IT (encrypted and signed value) 
// to address _to
// params: _to: the address to transfer to
//         _itCT: the encrypted value of the amount to transfer
//         _itSignature: the signature of the amount to transfer
//         revealRes: indicates if we should reveal the result of the transfer
// returns: In case revealRes is true, returns the result of the transfer. 
// In case revealRes is false, always returns true
<strong>function transfer(address _to, ctUint64 _itCT, bytes calldata _itSignature, 
</strong>                  bool revealRes) public returns (bool success){
    // Create IT from ciphertext and signature
    itUint64 memory it;
    it.ciphertext = _itCT;
    it.signature = _itSignature;
    // Verify the IT and transfer the value
    gtBool result = contractTransfer(_to, MpcCore.validateCiphertext(it));
    if (revealRes){
        return MpcCore.decrypt(result);
    } else {
        return true;
    }
}
</code></pre>

```solidity
2.
// Transfers the amount of tokens to address _to
// params: _to: the address to transfer to
//         _value: the value of the amount to transfer
//         revealRes: indicates if we should reveal the result of the transfer
// returns: In case revealRes is true, returns the result of the transfer. 
// In case revealRes is false, always returns true
function transfer(address _to, uint64 _value, bool revealRes) 
                  public returns (bool success){
    gtBool result = contractTransferClear(_to, _value);

    if (revealRes){
        return MpcCore.decrypt(result);
    } else {
        return true;
    }
}
```

```solidity
3. 
// Transfers the amount of tokens given inside the encrypted value to address _to
// params: _to: the address to transfer to
//         _value: the encrypted value of the amount to transfer
// returns: The encrypted result of the transfer.
function contractTransfer(address _to, gtUint64 _value) 
                          public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(msg.sender, _to);
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result) = 
                          MpcCore.transfer(fromBalance, toBalance, _value);

    emit Transfer(msg.sender, _to);
    setNewBalances(msg.sender, _to, newFromBalance, newToBalance);

    return result;
}
```

<pre class="language-solidity"><code class="lang-solidity"><strong>4. 
</strong><strong>// Transfers the amount of tokens to address _to
</strong>// params: _to: the address to transfer to
//         _value: the value of the amount to transfer
// returns: The encrypted result of the transfer.
function contractTransferClear(address _to, uint64 _value) 
                               public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(msg.sender, _to);
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result) = 
                               MpcCore.transfer(fromBalance, toBalance, _value);

    emit Transfer(msg.sender, _to, _value);
    setNewBalances(msg.sender, _to, newFromBalance, newToBalance);

    return result;
}
</code></pre>

## **Example Usage**

{% tabs %}
{% tab title="Python" %}
```python
plaintext_integer = 5
print("************* Transfer clear ", plaintext_integer, " to Alice *************")
# Transfer 5 SOD to Alice
function = contract.functions.transfer(alice_address.address, plaintext_integer, True)
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'transfer', INITIAL_BALANCE - plaintext_integer)

print("************* Transfer clear ", plaintext_integer, " to Alice *************")
# Transfer 5 SOD to Alice
function = contract.functions.contractTransferClear(alice_address.address, plaintext_integer)
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'contract transfer clear', INITIAL_BALANCE - 2*plaintext_integer)

print("************* Transfer IT ", plaintext_integer, " to Alice *************")
# In order to generate the input text, we need to use some data of the function. 
# For example, the address of the user, the address of the contract and also the function signature.
# To simplify the process of obtaining the function signature, we use a dummy function with placeholder inputs.
# After the signature is generated, we call prepare input text function and get the input text to use in the real function.
dummyCT = 0
dummySignature = bytes(65)
function = contract.functions.transfer(alice_address.address, dummyCT, dummySignature, False)
func_sig = get_function_signature(function.abi) # Get the function signature
# Prepare the input text for the function
ct, signature = prepare_IT(plaintext_integer, user_key, account, contract, func_sig, bytes.fromhex(private_key[2:]))
# Create the real function using the prepared IT
function = contract.functions.transfer(alice_address.address, ct, signature, False)
# Transfer 5 SOD to Alice
execute_and_check_balance(soda_helper, account, contract, function, user_key, 'transfer IT', INITIAL_BALANCE - 3*plaintext_integer)

```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const plaintext_integer = 5;
console.log("************* Transfer clear ", plaintext_integer, " to Alice *************");
// Transfer 5 SOD to Alice
let func = contract.methods.transfer(alice_address.address, plaintext_integer, true);
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - plaintext_integer)

console.log("************* Transfer clear ", plaintext_integer, " to Alice *************");
// Transfer 5 SOD to Alice
func = contract.methods.contractTransferClear(alice_address.address, plaintext_integer);
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - 2*plaintext_integer)

console.log("************* Transfer IT ", plaintext_integer, " to Alice *************")
// In order to generate the input text, we need to use some data of the function. 
// For example, the address of the user, the address of the contract and also the function signature.
// To simplify the process of obtaining the function signature, we use a dummy function with placeholder inputs.
// After the signature is generated, we call prepare input text function and get the input text to use in the real function.
const dummyCT = 0;
const dummySignature = Buffer.alloc(65);
func = contract.methods.transfer(alice_address.address, dummyCT, dummySignature, false); // Create dummy function to get the signature for prepare input text
let hashFuncSig = getFunctionSignature(func);
// After the function signature is obtained, prepare the input test for the function
let {ctInt, signature} = prepareIT(plaintext_integer, user_key, account.address, contract.options.address, hashFuncSig, Buffer.from(SIGNING_KEY.slice(2), 'hex'));
// Create the real function using the prepared IT
func = contract.methods.transfer(alice_address.address, ctInt, signature, false);
// Transfer 5 SOD to Alice
await executeAndCheckBalance(sodaHelper, func, contract, user_key, INITIAL_BALANCE - 3*plaintext_integer)
```
{% endtab %}
{% endtabs %}
