# 💲 Transfer

```solidity
function transfer(gtUint8 a, gtUint8 b, gtUint8 amount) internal returns (gtUint8, gtUint8, gtBool)

function transfer(gtUint8 a, gtUint8 b, uint8 amount) internal returns (gtUint8, gtUint8, gtBool)
```

The function transfers the value "amount" from "a" to "b".&#x20;

* If the value of "a" is less than the "amount", both "a" and "b" remain unchanged.&#x20;
* The "amount" can be either in clear (unencrypted) or encrypted form.&#x20;
* The function returns the new values of "a" and "b", along with a boolean value indicating whether the transfer was successful or not, encrypted for confidentiality.



## Usage example

Private ERC20 transfer function gets an encrypted value to transfer from the sender account to the receiver account.

```solidity
// Transfers the amount of tokens given inside the encrypted value to address _to
// params: _to: the address to transfer to
//         _value: the encrypted value of the amount to transfer
// returns: The encrypted result of the transfer.
function contractTransfer(address _to, gtUint64 _value) public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(msg.sender, _to);
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result) = MpcCore.transfer(fromBalance, toBalance, _value);

    emit Transfer(msg.sender, _to);
    setNewBalances(msg.sender, _to, newFromBalance, newToBalance);

    return result;
}
```

