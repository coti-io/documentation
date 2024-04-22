# 📨 Set Public

```solidity
function setPublic8(uint8 pt) internal returns (gtUint8) 
```

The function onboards the given clear input to the gcEVM, resulting in a Garbledtext™.



## Usage example

{% code lineNumbers="true" %}
```solidity
function getUserKeyTest(bytes calldata signedEK, bytes calldata signature, address addr) public returns (uint8) {

    gtUint8 a = MpcCore.setPublic8(uint8(5));
    gtUint8 c = MpcCore.add(a, uint8(5)); // 10
    userKey = MpcCore.getUserKey(signedEK, signature);
    ctUserKey = MpcCore.offBoardToUser(c, addr);
    ctUint8 ctSystemKey = MpcCore.offBoard(c);
    gtUint8 c1 = MpcCore.onBoard(ctSystemKey);
    x = MpcCore.decrypt(c1);
    return x;
}
```
{% endcode %}

The setPublic8 call in line 3 turn the clear value '5' to a Garbledtext™ object that can be used within the function's context.
