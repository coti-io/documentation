---
description: Never save garbledtext in storage
---

# 🆎 Proper Use of Types

## Never save `garbledtext` in storage

`Garbledtext` is an intermediate random value, It's crucial to understand that these values are transient, existing only for the duration of the transaction's execution and promptly deleted once the execution is completed. Storing such a type will cause a loss of data as it has no meaning once the execution is completed.\


<mark style="color:red;">**Don't**</mark>

```solidity
contract BadContract {
  gtUint64 private storedGarbledText;

  function setZero() public{
    storedGarbledText = mpcCore.SetPublic64(0);
  }
}  
```

[<mark style="color:blue;">**Do**</mark>](#user-content-fn-1)[^1]

```solidity
contract GoodContract {
  ctUint64 private storedCipherText;
                                                
  function setZero() public {
    gtUint64 gtZero = mpcCore.SetPublic64(0);
    storedCipherText = mpcCore.offboard(gtZero);  
  }
}
```

## Never Pass `Ciphertext` as Parameter to Another Contract or As User Input

Sending a `ciphertext` from an Externally Owned Account (EOA) will fail unless it is accompanied by the correct signature to pass validation. Moreover, transmitting ciphertext between different contracts renders it invalid within the scope of the receiving contract and thus ineffective.

<mark style="color:red;">**Don't**</mark>

```solidity
contract BadContract {
  ctUint8 private ctVal;
  
  function passCiphertext(ctUint8 val) public {
   ctVal = val;
  }
}
```

[<mark style="color:blue;">**Do**</mark>](#user-content-fn-2)[^2]

```solidity
contract GoodContract {
  ctUint8 private ctDouble;
  
  function passGarbledtext(gtUint8 val) public {
   gtUint8 gtDouble = MpcCore.add(val, val);
   ctDouble = MpcCore.offBoard(gtDouble)
  }
}
```

<mark style="color:blue;">**Or**</mark>

```solidity
contract GoodContract {
  ctUint8 private ct;
  
  function passInputtext(ctUint64 _itCT, bytes calldata _itSignature) public {
    itUint64 memory it;
    it.ciphertext = _itCT;
    it.signature = _itSignature;
    // validate the IT and
    gtUserInput = MpcCore.validateCiphertext(it);
    ct = MpcCore.offBoard(gtUserInput )
  }
}
```

## Never Return a `Ciphertext`  to Another Contract

Returning a `ciphertext` for future use in a different contract is not feasible, rendering such an action pointless. However, it is logical to return such a type to an Externally Owned Account (EOA) if it was specifically designated for that particular user.

<mark style="color:red;">**Don't : do not return system encrypted ciphertext**</mark>

```solidity
contract BadContract {
  ctUint8 private ctVal;
  
  function passCiphertext(ctUint8 val) public returns (ctUint16){
   return ctVal;
  }
}
```

[<mark style="color:blue;">**Do**</mark>](#user-content-fn-3)[^3] <mark style="color:blue;">**: return ciphertext to a specific user**</mark>

<pre class="language-solidity"><code class="lang-solidity"><strong>contract GoodContract {
</strong>  function balanceOf() public returns (ctUint64 balance){
    ctUint64 balance = balances[msg.sender];
    // The balance is saved encrypted using the system key. However, to allow 
    // the user to access it, the balance needs to be re-encrypted using the user key. 
    // Therefore, we decrypt the balance (onBoard) and then encrypt it again using 
    // the user key (offBoardToUser).
    gtUint64 balanceGt = MpcCore.onBoard(balance);
    return MpcCore.offBoardToUser(balanceGt, msg.sender);
  }
}
</code></pre>

[<mark style="color:blue;">**Do**</mark>](#user-content-fn-4)[^4] <mark style="color:blue;">**: return**</mark><mark style="color:blue;">** **</mark><mark style="color:blue;">**`garbledtext`**</mark><mark style="color:blue;">** **</mark><mark style="color:blue;">**to a contract**</mark>

```solidity
contract GoodContract {
  function contractTransfer(address _to, gtUint64 _value) public returns (gtBool success){
    (gtUint64 fromBalance, gtUint64 toBalance) = getBalances(msg.sender, _to);
    (gtUint64 newFromBalance, gtUint64 newToBalance, gtBool result) = MpcCore.transfer(fromBalance, toBalance, _value);
  
    emit Transfer(msg.sender, _to);
    setNewBalances(msg.sender, _to, newFromBalance, newToBalance);
  
    return result;
   }
}
```

[^1]: 

[^2]: 

[^3]: 

[^4]: 
