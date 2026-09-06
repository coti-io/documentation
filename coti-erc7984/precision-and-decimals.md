# Precision and decimals

PoD confidential wrappers are 1:1 collateralised. Lock WETH, get `p.ETH`. Lock USDC, get `p.USDC`. The private token mirrors the underlying — same decimals, same supply, same value — and unwraps back on demand.

That is a **precision vs FHE-cost** choice on Zama, not a hard platform bit-width ceiling.

## What the Zama token reference actually uses

OpenZeppelin [`ERC7984`](https://docs.openzeppelin.com/confidential-contracts/token) (`@openzeppelin/confidential-contracts` 0.5.3) stores balances as **`euint64`** and implements `decimals()` as **`6`**. ERC-7984 itself only requires a plaintext `decimals()`; it does not mandate 64-bit math. `euint64` is the reference-design type.

`uint64` max is \(2^{64}-1\) ≈ \(1.84 \times 10^{19}\):

| Decimals | Max whole tokens in `euint64` | Fit with 18-decimal collateral |
| -------: | ----------------------------: | :----------------------------- |
| 18 | ≈ **18.45** | Does not fit a 1:1 WETH wrapper above ~18 ETH |
| 6 | ≈ **18.45 trillion** | Fits large supplies; **12 digits of token precision are dropped** vs 18-decimal ERC-20 math |

Zama confidential wrappers for 18-decimal underlyings scale into 6 decimals (for example \(10^{12}\) on mock WETH). Arithmetic on 6-decimal amounts does not match 18-decimal ERC-20 rounding and dust behaviour.

## `euint128` and `euint256` are not a silent 64-bit wall

[FHEVM types](https://docs.zama.org/protocol/solidity-guides/smart-contract/types) (`@fhevm/solidity` 0.13.3):

- **`euint64`** and **`euint128`** support the main arithmetic operators (add, sub, mul, comparisons, `select`, …).
- **`euint256`** supports bitwise ops, equality, `select`, and rand — **not** add/mul. It is not a drop-in confidential `uint256` balance type.

[HCU](https://docs.zama.org/protocol/solidity-guides/development-guide/hcu) for non-scalar ops (higher = more coprocessor work):

| Op | `euint64` | `euint128` |
| :- | --------: | ---------: |
| `add` | 162,000 | 259,000 |
| `mul` | 596,000 | 1,686,000 |

A confidential 18-decimal WETH-style balance **can** be represented in `euint128` (\(2^{128}-1 / 10^{18}\) is far above any realistic ETH supply). The reference ERC-7984 token does not do that: it keeps `euint64` and 6 decimals so FHE cost stays lower. The tradeoff is **precision (and ERC-20 decimal compatibility) versus FHE cost**, not “the platform cannot exceed ~18 ETH.”

## COTI

COTI carries **256-bit** encrypted integers end to end (`itUint256` / `gtUint256` / `ctUint256`). PoD wrappers keep the underlying’s decimals. Four of the six live pTokens are 18-decimal, including `p.ETH` and `p.AVAX`.

The [deployed contracts](deployed-contracts.md) page lists the decimals of each live pToken.
