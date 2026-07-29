# Contract Addresses

Addresses for **COTI Testnet (7082400)** and **COTI Mainnet (2632500)** below match the Privacy Portal source of truth: `coti-privacy-portal/src/contracts/config.ts`. **Sepolia** and **Avalanche Fuji** PoD Privacy Portal addresses match `@coti-io/coti-wallet-plugin` / PEI `deployConfig.json`. They may change after contract redeploys; always verify against those sources or your deployed environment.

#### COTI Testnet (Chain ID: 7082400)

| Token                | Address                                      |
| -------------------- | -------------------------------------------- |
| p.COTI (PrivateCoti) | `0x6cE8907414986E73De9e7D28d62Ea2080F8E88E1` |
| p.WETH               | `0xF009BADb181d471995a1CFF406C3Db7B180F64eA` |
| p.WBTC               | `0xB50F1680a4C69145ABc09A2A71c8D5b8051578cF` |
| p.USDT               | `0xcEF137E96eDF68EE99D4CdEa7085f154d74895cD` |
| p.USDC.e             | `0x37f78dcCd15876F74391EF1F01b76557D9FF1dea` |
| p.WADA               | `0x1245f50a3E9129A219b4bf66D10fEaEA47467B69` |
| p.gCOTI              | `0x1503b02a4Aa27812306c65116FD23b733603F142` |

#### COTI Testnet — Public Tokens

| Token  | Address                                      |
| ------ | -------------------------------------------- |
| WETH   | `0x8bca4e6bbE402DB4aD189A316137aD08206154FB` |
| WBTC   | `0x5dBDb2E5D51c3FFab5D6B862Caa11FCe1D83F492` |
| USDT   | `0x9e961430053cd5AbB3b060544cEcCec848693Cf0` |
| USDC.e | `0x63f3D2Cc8F5608F57ce6E5Aa3590A2Beb428D19C` |
| WADA   | `0xe3E2cd3Abf412c73a404b9b8227B71dE3CfE829D` |
| gCOTI  | `0x878a42D3cB737DEC9E6c7e7774d973F46fd8ed4C` |

#### COTI Testnet — Bridges

| Bridge                  | Address                                      |
| ----------------------- | -------------------------------------------- |
| PrivacyBridgeCotiNative | `0xb8Bb4fe953eAa53D528FAc95C1d9955B2b60D582` |
| PrivacyBridgeWETH       | `0x1841071A0296364739370a6d2F64c0eE46361fA0` |
| PrivacyBridgeWBTC       | `0x362faD66210401ADfAf27B98776F1e8D21dfc529` |
| PrivacyBridgeUSDT       | `0x73116aa5a50cADca47FD03Ca0B80D133346442FA` |
| PrivacyBridgeUSDCe      | `0x9C92Ad40553758C3d11Dcd8495Ee0ce3fd8fE0A1` |
| PrivacyBridgeWADA       | `0x3cB6e1E9cd504669DAb49910c30cDAfA8D05B641` |
| PrivacyBridgegCOTI      | `0x8A6ca3984Cb187f90C9Bd24c71C70eF97A71A8fA` |

#### COTI Mainnet (Chain ID: 2632500)

Mainnet **native private COTI** bridge and **public** token addresses from the same portal config:

| Role / token              | Address                                      |
| ------------------------- | -------------------------------------------- |
| WETH                      | `0x639aCc80569c5FC83c6FBf2319A6Cc38bBfe26d1` |
| WBTC                      | `0x8C39B1fD0e6260fdf20652Fc436d25026832bfEA` |
| USDT                      | `0xfA6f73446b17A97a56e464256DA54AD43c2Cbc3E` |
| USDC.e                    | `0xf1Feebc4376c68B7003450ae66343Ae59AB37D3C` |
| WADA                      | `0xe757Ca19d2c237AA52eBb1d2E8E4368eeA3eb331` |
| gCOTI                     | `0x7637C7838EC4Ec6b85080F28A678F8E234bB83D1` |

#### PoD Privacy Portal — Sepolia (Chain ID: 11155111)

Shared PoD Inbox: `0x3b8B70819f27e0438cBcE7f31894f799da52648F`.

| Contract | Address |
| --- | --- |
| Privacy Portal factory | `0x0117d640ce96805739cf5f82683b0dd9532541ee` |
| Portal implementation | `0x7f36e1eabbee6a1cd724b4ade37fe475f807d982` |
| Pod token implementation | `0xe8e2fdd23ea2d5f9bb4632d11f7267602a059e5d` |
| PoD price oracle | `0x3281160888138e786c3eb0f4f4cc51453d8dfeff` |

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| pMTT | `0xd3f5c63f4D87D2235b295FbA83351d31d0eD1BeE` | `0x621E744eF059262Fd531a0f345d38Ce31d92D105` | `0x1566ADA98695D39b2D5A8e1359d7Af9D567c74ab` |
| pUSDC | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` | `0x79679CE36664c3b1360501B2c7ea6bbee65a2717` | `0xc04Cb7256E849C34877D801A77f9165BaC209c06` |
| pWETH | `0x7b79995e5f793A07Bc00c21412e50Ecae098E7f9` | `0x7666F6576956530E2D56CDB548b71e62286d1d18` | `0xD586736543F7666d1adbF862B769Ba838a9a3deD` |

#### PoD Privacy Portal — Avalanche Fuji (Chain ID: 43113)

See also [Avalanche Fuji network addresses](../../privacy-on-avalanche/networks/fuji.md#privacy-portal-fuji).

| Contract | Address |
| --- | --- |
| Privacy Portal factory | `0xf3cf653e1baee7b4e4001067780dee38991b1cbd` |
| Portal implementation | `0x63e97937e42c153cdeb25e9aca9d3d0373aec0a5` |
| Pod token implementation | `0xa7e4838327317f4ce6cc8b5ab07a57fdba842c77` |
| PoD price oracle | `0xf2283ca93a6747c547a961c50d0393d549c57268` |

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| pMTT | `0x328e70e1c52662cd5f19f824fcb8b463d77a6686` | `0xf4100d21eB4B1a66aDde58A01D1E32356F268b3F` | `0xFC6283a9000d7D5Cf8A058A04A9ED90265Af1634` |
| pUSDC | `0x5425890298aed601595a70AB815c96711a31Bc65` | `0x090D2dc8C38275939b9381Ff2aa53012Ff412E34` | `0xe2235E064a3CEB5F1765c3b095855549d3c8A8a4` |
| pWAVAX | `0xd00ae08403B9bbb9124bB305C09058E32C39A48c` | `0x20e7239cd78BDf2E8f34c52947e54fE68D7b536F` | `0x0c58954d91392794A50F610dF8c84228D63BE9D4` |

### Token Decimals Reference

| Token                    | Decimals |
| ------------------------ | -------- |
| p.COTI, p.WETH, p.gCOTI  | 18       |
| p.WBTC                   | 8        |
| p.USDT, p.USDC.e, p.WADA | 6        |
