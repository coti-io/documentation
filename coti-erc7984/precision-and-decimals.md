# Precision and decimals

## Wrap the assets that actually exist

PoD ERC-7984 is a 1:1 collateralised confidential wrapper. Lock WETH, get `p.ETH`. Lock USDC, get `p.USDC`. The private token mirrors the underlying exactly — same decimals, same supply, same value — and unwraps back on demand.

That "same decimals" part is where PoD stands alone.

**FHE-based confidential token standards store balances as 64-bit encrypted integers.** At 18 decimals, a 64-bit ceiling caps a token at roughly **18.4 whole units** before it overflows. A confidential 1:1 WETH wrapper is not difficult under that constraint — it is arithmetically impossible above ~18 ETH.

PoD carries **full 256-bit precision** end to end: 18 decimals, `uint256` range, no ceiling worth naming. Four of the six pTokens live today are 18-decimal, including `p.ETH` and `p.AVAX`.

> **If you want to wrap real liquidity confidentially, 256-bit is not a preference. It is the entry requirement.**

The [deployed contracts](deployed-contracts.md) page lists the decimals of each live pToken.
