# 🆗 Approve

The function approves a new allowance for the spender.

There are three types of approve:

```solidity
1. function approve(address _spender, ctUint64 _itCT, bytes calldata _itSignature) public returns (bool success)
```

```solidity
2. function approve(address _spender, gtUint64 _value) public returns (bool success)
```

```solidity
3. function approveClear(address _spender, uint64 _value) public returns (bool success)
```

* 1: Validates the inputText to get the required amount to allow.
* 3: Allow an encrypted amount. This version is used by a contract that already has the value as encrypted gtUint64.

**Parameters:**&#x20;

* \_spender: The address to allow the given amount
* \_itCT, \_itSignature: An encrypted value to allow as an InputText&#x20;
* \_value: The value to allow (clear or encrypted depends on the type)

**Return value:**&#x20;

* Always true

## Full Solidity Code

```solidity
1. 
// Sets the new allowance given inside the IT (encrypted and signed value) of the 
// spender
function approve(address _spender, ctUint64 _itCT, bytes calldata _itSignature) 
                 public returns (bool success){
    // Create IT using the given CT and signature
    itUint64 memory it;
    it.ciphertext = _itCT;
    it.signature = _itSignature;
    return approve(_spender, MpcCore.validateCiphertext(it));
}
```

```solidity
2.
// Sets the new encrypted allowance of the spender
function approve(address _spender, gtUint64 _value) public returns (bool success){
    address owner = msg.sender;
    setApproveValue(owner, _spender, MpcCore.offBoard(_value));
    emit Approval(owner, _spender);
    return true;
}
```

```solidity
// Sets the new allowance of the spender
function approveClear(address _spender, uint64 _value) public returns (bool success){
    address owner = msg.sender;
    gtUint64 gt = MpcCore.setPublic64(_value);
    setApproveValue(owner, _spender, MpcCore.offBoard(gt));
    emit Approval(owner, _spender, _value);
    return true;
}
```

## **Example Usage:**

{% tabs %}
{% tab title="Python" %}
```python
print("************* Approve ", plaintext_integer*10, " to my address *************")
# Set allowance for this account
function = contract.functions.approveClear(account.address, plaintext_integer*10)
soda_helper.call_contract_function_transaction("private_erc20", function)
print("************* Check my allowance *************")
# Get my allowance to check that the allowance is correct
check_allowance(account, contract, user_key, plaintext_integer*10)

print("************* Approve IT 50 to my address *************")
# Approve 50 SOD to this account
function = contract.functions.approve(account.address, dummyCT, dummySignature) # Dummy function
func_sig = get_function_signature(function.abi) # Get the function signature
# Prepare the input text for the function
ct, signature = prepare_IT(50, user_key, account, contract, func_sig, bytes.fromhex(private_key[2:]))
function = contract.functions.approve(account.address, ct, signature)
soda_helper.call_contract_function_transaction("private_erc20", function)
print("************* Check my allowance *************")
# Check that the allowance has changed to 50 SOD
check_allowance(account, contract, user_key, 50)
```
{% endtab %}

{% tab title="JavaScript" %}
<pre class="language-javascript"><code class="lang-javascript">console.log("************* Approve ", plaintext_integer*10, " to my address *************")
<strong>// Set allowance for this account
</strong>func = contract.methods.approveClear(account.address, plaintext_integer*10)
await sodaHelper.callContractFunctionTransaction(func);
console.log("************* Check my allowance *************")
// Get my allowance to check that the allowance is correct
checkAllowance(account, contract, user_key, plaintext_integer*10)

console.log("************* Approve IT 50 to my address *************")
// Approve 50 SOD to this account
func = contract.methods.approve(account.address, dummyCT, dummySignature); // Dummy function to get the signature
hashFuncSig = getFunctionSignature(func);
({ctInt, signature} = prepareIT(50, user_key, account.address, contract.options.address, hashFuncSig, Buffer.from(SIGNING_KEY.slice(2), 'hex')));
func = contract.methods.approve(account.address, ctInt, signature);
await sodaHelper.callContractFunctionTransaction(func);
console.log("************* Check my allowance *************")
// Check that the allowance has changed to 50 SOD
checkAllowance(account, contract, user_key, 50);
</code></pre>
{% endtab %}
{% endtabs %}
