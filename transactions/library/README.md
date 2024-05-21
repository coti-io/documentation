# Library

Our solidity library provides methods for manipulating secret data. It's essential to utilize these functions for any operations involving sensitive information. Additionally, it's worth noting that you can also call these functions with some of the inputs being scalars. Below are the function signatures for the available operations. Note that different type sizes can be mixed during calls.

A user wishing to invoke operations on private data should import this library in `MpcCore.sol`

The following list pertains to a single type size, as the signatures remain consistent across all type sizes and for mixed-size calls.

## Secret Boolean type and various operations

#### (please note this is a small subset, for full reference see [MpcCore.sol](https://github.com/coti-io/confidentiality-contracts/blob/main/contracts/lib/MpcCore.sol))

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

## Secret type of 32 bits and various operations&#x20;

#### (see [MpcInterface.sol](https://github.com/coti-io/confidentiality-contracts/blob/main/contracts/lib/MpcInterface.sol))

```solidity
function OnBoard(bytes1 metaData, uint256 ct) external returns (uint256 result);
function OffBoard(bytes1 metaData, uint256 ct) external returns (uint256 result);
function OffBoardToUser(bytes1 metaData, uint256 ct, bytes calldata addr) external returns (uint256 result);
function SetPublic(bytes1 metaData, uint256 ct) external returns (uint256 result);
function Rand(bytes1 metaData) external returns (uint256 result);
function RandBoundedBits(bytes1 metaData, uint8 numBits) external returns (uint256 result);
function Add(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Sub(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Mul(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Div(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Rem(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function And(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Or(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Xor(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Shl(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Shr(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Eq(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Ne(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Ge(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Gt(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Le(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Lt(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Min(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Max(bytes3 metaData, uint256 lhs, uint256 rhs) external returns (uint256 result);
function Decrypt(bytes1 metaData, uint256 a) external returns (uint256 result);
function Mux(bytes3 metaData, uint256 bit, uint256 a,uint256 b) external returns (uint256 result);
function Not(bytes1 metaData, uint256 a) external returns (uint256 result);
function Transfer(bytes4 metaData, uint256 a, uint256 b, uint256 amount) external returns (uint256 new_a, uint256 new_b, uint256 res);
function TransferWithAllowance(bytes4 metaData, uint256 a, uint256 b, uint256 amount, uint256 allowance) external returns (uint256 new_a, uint256 new_b, uint256 res, uint256 new_allowance);
function ValidateCiphertext(bytes1 metaData, uint256 ciphertext, bytes calldata signature) external returns (uint256 result);
function GetUserKey(bytes calldata signedEK) external view returns (bytes memory encryptedKey);

```
