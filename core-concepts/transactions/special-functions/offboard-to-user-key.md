# Offboard To User Key

```solidity
function offBoardToUser(gtUint8 pt, address addr) internal returns (ctUint8)
```

The function offboards the given Garbledtext™ from the gcEVM, resulting in a Ciphertext.

The offboarding process uses the **user AES key** associated with the given address to encrypt the value inside the Garbledtext™.

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

The offBoardToUser call in line 6 turns the "c" Garbledtext™ value to a Ciphertext.
