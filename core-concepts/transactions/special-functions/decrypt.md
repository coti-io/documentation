# 🌟 Decrypt

```solidity
function decrypt(gtUint8 ct) internal returns (uint8)
```

Decrypt function returns the clear value of the given Ciphertext.

## Usage example

```solidity
function randomExample() public {
    gtUint8 a = MpcCore.setPublic8(uint8(5));
    gtUint8 random = MpcCore.rand8();
    
    gtBool bit = MpcCore.le(a,random);  // Check if a <= random
    gtUint8 smaller = MpcCore.mux(bit,a,random);
    
    uint8 res = MpcCore.decrypt(smaller);
    returns res;
}
```

The decrypt call in the last line decrypts the 'smaller" value and returns it on the clear.
