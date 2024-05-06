# Balance Of

```solidity
    // The function returns the encrypted account balance utilizing the user's 
    // secret key. 
    // Since the balance is initially encrypted internally using the system's AES 
    // key, the user cannot access it. 
    // Thus, the balance undergoes re-encryption using the user's secret key. 
    // As a result, the function is not designated as a "view" function.
    function balanceOf() public returns (ctUint64 balance){
        ctUint64 balance = balances[msg.sender];
        // The balance is saved encrypted using the system key. 
        // However, to allow the user to access it, the balance needs to be 
        // re-encrypted using the user key. 
        // Therefore, we decrypt the balance (onBoard) and then encrypt it again using 
        // the user key (offBoardToUser).
        gtUint64 balanceGt = MpcCore.onBoard(balance);
        return MpcCore.offBoardToUser(balanceGt, msg.sender);
    }
```

The function returns the balance of the calling account.

**Return value:**

The balance of the calling account encrypted using his AES key.

**Example Usage:**

{% tabs %}
{% tab title="Python" %}
```python
my_CTBalance = contract.functions.balanceOf().call({'from': account.address})
my_balance = decrypt_value(my_CTBalance, user_key)
```
{% endtab %}
{% endtabs %}
