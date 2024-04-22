# ❓ Random

```solidity
function rand8() internal returns (gtUint8)

function randBoundedBits8(uint8 numBits) internal returns (gtUint8)
```

`Rand` function generates an encrypted random value

`RandBoundedBits8` function generates an encrypted random value that falls within the range of \[0, 2^numBits]



## Usage example

```solidity
function randomExample() public {
gtUint8 a = MpcCore.setPublic8(uint8(5));
gtUint8 random = MpcCore.rand8();

gtBool bit = MpcCore.le(a,random);  // Check if a <= random
gtUint8 smaller = MpcCore.mux(bit,a,random);

uint8 res = MpcCore.decrypt(smaller);
}
```

