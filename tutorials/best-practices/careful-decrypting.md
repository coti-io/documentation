# Careful Decrypting

The creator of a contract must understand the implications of decryption in various scenarios. For instance, decryption should never occur within a view function without considering alternative security measures, as relying solely on `msg.sender` can be unreliable due to the potential for forging.

As a general precaution, handling a call to decrypt should be approached with caution, considering the various options for invoking such a function. For example, a secret intended for a specific user should not be decrypted in a manner that exposes it to everyone.

<mark style="color:red;">**Don't: Do not reveal data intended for a specific user to everybody**</mark>

```solidity
contract BadContract {
  //...
  function balanceOf() public view returns (uint64 balance){
    ctUint64 balance = balances[msg.sender];
   
    gtUint64 balanceGt = MpcCore.onBoard(balance);
    
    // SHOULD NEVER BE CALLED
    // A SIMPLE CALL TO GETBALANCE REVEALS THE BALANCE TO EVERYBODY
    return MpcCore.decrypt(balanceGt);
  }
  //...
}
```

<mark style="color:blue;">**Do**</mark> <mark style="color:blue;">:</mark> <mark style="color:blue;">`Offboard`</mark> <mark style="color:blue;">to a specific user.</mark>

<pre class="language-solidity"><code class="lang-solidity"><strong>contract GoodContract {
</strong><strong>   //...
</strong>  function balanceOf() public returns (ctUint64 balance){
    ctUint64 balance = balances[msg.sender];
    // The balance is saved encrypted using the system key. However, to allow 
    // the user to access it, the balance needs to be re-encrypted using the user key. 
    // Therefore, we decrypt the balance (onBoard) and then encrypt it again using 
    // the user key (offBoardToUser).
    gtUint64 balanceGt = MpcCore.onBoard(balance);
    return MpcCore.offBoardToUser(balanceGt, msg.sender);
  }
  //...
}
</code></pre>
