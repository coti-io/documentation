# 📢 No Public Contract Variables

Public state variables are inherently accessible to other contracts for manipulation. If a different contract modifies these values, they may not undergo the necessary validation process. Consequently, such modifications may result in an invalid ciphertext.

<mark style="color:red;">**Don't**</mark><mark style="color:red;">: Do not use</mark> <mark style="color:red;"></mark><mark style="color:red;">`public`</mark> <mark style="color:red;"></mark><mark style="color:red;">secret types.</mark>

```solidity
contract BadContract {
  ctUint32 public bad;
  //....
}
```

<mark style="color:blue;">**Do**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">: Remove the</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`public`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">keyword and either add</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`private`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">or retain the default</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`internal`</mark><mark style="color:blue;">.</mark>
