# 📤 Onboard

```solidity
function onBoard(ctUint8 ct) internal returns (gtUint8)
```

The function onboards a given Ciphertext to the gcEVM, resulting in a Garbledtext™ value.

{% hint style="info" %}
Please note that the function `onboard` can only be invoked with ciphertext encrypted by the system AES key. Such ciphertexts are generated through the `offboard` call. If `offboardToUser` is called, encrypting the data with the user key and then attempting to onboard it using the `onboard` function will result in an error.

In order to onboard a ciphertext encrypted by the user AES key, use the validate Input text function.
{% endhint %}

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

The onBoardcall in line 8 turns the "ctSystemKey" Ciphertext value to a Garbledtext™.
