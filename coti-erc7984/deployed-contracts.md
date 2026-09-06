# Deployed contracts

Six PoD pTokens on public testnets (addresses as of this documentation; redeploys can change them). These are **COTI confidential wrappers**, not ERC-7984 deployments.

**Avalanche Fuji**

| Token    | Address                                      | Decimals |
| :------- | :------------------------------------------- | :------: |
| `p.MTT`  | `0x02f284a1968160E1d3e4bC2BA3261be49725E765` |    18    |
| `p.USDC` | `0x21576D8CCE47d044C5815bd59eca1F6DA94c65A5` |    6     |
| `p.AVAX` | `0x74d47cD68203066c97BA99787Fe1e0c68Ce42b04` |    18    |

**Ethereum Sepolia**

| Token    | Address                                      | Decimals |
| :------- | :------------------------------------------- | :------: |
| `p.MTT`  | `0x0510F0b32828D5fB472dE5A5bE30b370c5D1a056` |    18    |
| `p.USDC` | `0xD7B3D49F85000489708B7db5B0f1a8693Fc707f3` |    6     |
| `p.ETH`  | `0xd33A363459c6Ee0C4F8504E380E8D3Aa4F209116` |    18    |

Each pair is a `PrivacyPortalFactory` minimal-proxy clone: one portal and one pToken per asset. Decimals match the underlying (see [Precision and decimals](precision-and-decimals.md)).
