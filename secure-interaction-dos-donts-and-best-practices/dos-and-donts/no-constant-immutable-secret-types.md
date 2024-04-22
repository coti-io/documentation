# ❌ No Constant/Immutable Secret Types

In Solidity, `constant` variables are evaluated at compile-time and replaced with their respective values in the bytecode of the contract. On the other hand, `immutable` variables are set at contract deployment and cannot be changed thereafter. In both cases, these scenarios prevent them from being recognized by our security mechanism, rendering them invalid.

<mark style="color:red;">**Don't**</mark><mark style="color:red;">: use constant\immutable secret types.</mark>

```solidity
contract BadContract {
  ctUint32 private constant constVal = MpcCore.setPublic32(5);
  ctUint32 private immutable immutableVal;

  constructor(uint32 _val) {
    immutableVal = MpcCore.setPublic32(_val);
  }
}
```

<mark style="color:blue;">**Do**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">: Simply remove the constant\immutable keywords</mark>
