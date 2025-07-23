# Check Overflow

Extra caution should be exercised when performing addition, subtraction, or division operations. For instance, it's important to verify that an overflow hasn't occurred in cases where it could potentially happen.

<mark style="color:red;">**Avoid:**</mark>

```solidity
contract AvoidContract {
    function addinGtWithoutChecking(gtUint16 lhs, gtUint16 rhs) public {
        gtUint16 addResult = MpcCore.add(lhs, rhs);
        return addResult;
    }
}
```

<mark style="color:blue;">**Do**</mark> <mark style="color:blue;">:</mark> <mark style="color:blue;">Check that the result is greater than one of the operands, return zero for example on overflow</mark>

```solidity
contract AvoidContract {
    function addinGtReturnZeroOnOverFlow(gtUint16 lhs, gtUint16 rhs) public {
        gtUint16 tempAddResult = MpcCore.add(lhs, rhs);
        gtBool isOverflowed = MpcCore.lt(tempAddResult , lhs);
        gtUint16 gtZero = MpcCore.setPublic16(0);
        addResult = MpcCore.mux(isOverflow, gtZero, tempAddResult);
        return addResult;
    }
}
```

{% hint style="info" %}
You may also use the `checkedAdd`, `checkedSub` and `checkedMul` operands, which cause the transaction to revert on overflow/underflow.
{% endhint %}
