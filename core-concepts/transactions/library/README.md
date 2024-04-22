# Library

Our solidity library provides methods for manipulating secret data. It's essential to utilize these functions for any operations involving sensitive information. Additionally, it's worth noting that you can also call these functions with some of the inputs being scalars. Below are the function signatures for the available operations. Note that different type sizes can be mixed during calls.

A user wishing to invoke operations on private data should import this library in `MpcCore.sol`

The following list pertains to a single type size, as the signatures remain consistent across all type sizes and for mixed-size calls.\\

## Secret Boolean type and various operations.

```solidity
// Boolean secret type
function validateCiphertext(itBool memory input) internal returns (gtBool);
function onBoard(ctBool ct) internal returns (gtBool);
function offBoard(gtBool pt) internal returns (ctBool);
function offBoardToUser(gtBool pt, address addr) internal returns (ctBool);
function setPublic(bool pt) internal returns (gtBool);
function rand() internal returns (gtBool);
function and(gtBool a, gtBool b) internal returns (gtBool);
function or(gtBool a, gtBool b) internal returns (gtBool);
function xor(gtBool a, gtBool b) internal returns (gtBool);
function eq(gtBool a, gtBool b) internal returns (gtBool);
function ne(gtBool a, gtBool b) internal returns (gtBool);
function decrypt(gtBool ct) internal returns (bool);
function mux(gtBool bit, gtBool a, gtBool b) internal returns (gtBool);
function not(gtBool a) internal returns (gtBool);

```

## Secret type of 32 bits and various operations.

```solidity
// 32 bit operations 
function validateCiphertext(itUint32 memory input) internal returns (gtUint32);
function onBoard(ctUint32 ct) internal returns (gtUint32);
function offBoard(gtUint32 pt) internal returns (ctUint32);
function offBoardToUser(gtUint32 pt, address addr) internal returns (ctUint32);
function setPublic32(uint32 pt) internal returns (gtUint32);
function rand32() internal returns (gtUint32);
function randBoundedBits32(uint8 numBits) internal returns (gtUint32);
function add(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function sub(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function mul(gtUint32 a, gtUint32 b) internal returns (gtUint64);
function div(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function rem(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function and(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function or(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function xor(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function shl(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function shr(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function eq(gtUint32 a, gtUint32 b) internal returns (gtBool);
function ne(gtUint32 a, gtUint32 b) internal returns (gtBool);
function ge(gtUint32 a, gtUint32 b) internal returns (gtBool);
function gt(gtUint32 a, gtUint32 b) internal returns (gtBool);
function le(gtUint32 a, gtUint32 b) internal returns (gtBool);
function lt(gtUint32 a, gtUint32 b) internal returns (gtBool);
function min(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function max(gtUint32 a, gtUint32 b) internal returns (gtUint32);
function decrypt(gtUint32 ct) internal returns (uint32);
function mux(gtBool bit, gtUint32 a, gtUint32 b) internal returns (gtUint32);
function transfer(gtUint32 a, gtUint32 b, gtUint32 amount) internal returns (gtUint32, gtUint32, gtBool);

// 32 bit add example with scalars
function add(gtUint32 a, uint32 b) internal returns (gtUint32);
function add(uint32 a, gtUint32 b) internal returns (gtUint32);

// 32 bit add example with different sized types
function add(gtUint8 a, gtUint32 b) internal returns (gtUint32);
function add(gtUint32 a, gtUint8 b) internal returns (gtUint32);
function add(gtUint16 a, gtUint32 b) internal returns (gtUint32);
function add(gtUint32 a, gtUint16 b) internal returns (gtUint32);

```
