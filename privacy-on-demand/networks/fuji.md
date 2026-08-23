# Avalanche Fuji

Avalanche Fuji is a supported **host chain** for Privacy on Demand. Your dApp contracts and the local **Inbox** live here; private computation still runs on [COTI Testnet](coti-testnet.md).

> **Note:** Addresses below match `pod-ecosystem-integration/deployConfig.json` (Inbox salt `pod.inbox.v2.2`). They may change after contract redeploys. Confirm against that file or your SDK release before production use.

## Network details

| Parameter | Value |
| --- | --- |
| Network name | Avalanche Fuji C-Chain |
| Chain ID | `43113` |
| Currency | AVAX |
| RPC URL | `https://api.avax-test.network/ext/bc/C/rpc` |
| Block explorer | [https://testnet.snowscan.xyz](https://testnet.snowscan.xyz) |

Private execution for Fuji dApps targets **COTI Testnet** (`7082400`). See [COTI Testnet](coti-testnet.md) for the MPC executor and COTI-side Inbox.

## PoD contracts

| Contract | Address | Description |
| --- | --- | --- |
| Inbox | [`0x3b8B70819f27e0438cBcE7f31894f799da52648F`](https://testnet.snowscan.xyz/address/0x3b8B70819f27e0438cBcE7f31894f799da52648F) | Cross-chain message router (CREATE3; same address on every PoD chain) |
| Price oracle | [`0x95ce33378c88734f3d86b51a4c6dc588722995fd`](https://testnet.snowscan.xyz/address/0x95ce33378c88734f3d86b51a4c6dc588722995fd) | Local/remote token prices used by Inbox fee conversion |
| MpcAdder (example) | [`0x8b7d9e70477aabe68500b72acb7f367993edde39`](https://testnet.snowscan.xyz/address/0x8b7d9e70477aabe68500b72acb7f367993edde39) | Reference primitive-only adder dApp on Fuji |

## PoD cross-chain Privacy Portal (Fuji)

This is the **PoD host-chain Privacy Portal** (factory + pTokens on Fuji, private compute via COTI Inbox). It is **not** the [native COTI Privacy Portal / PrivateERC20](../../coti-privacy-portal/README.md) that runs entirely on COTI.

| Contract | Address |
| --- | --- |
| Privacy Portal factory | [`0xaf9327277cb370d536d2c8a9e15a0a7ff6c42c15`](https://testnet.snowscan.xyz/address/0xaf9327277cb370d536d2c8a9e15a0a7ff6c42c15) |
| Portal implementation | [`0xf4fb32758e41c1b465049c5c967a6604b6c28dce`](https://testnet.snowscan.xyz/address/0xf4fb32758e41c1b465049c5c967a6604b6c28dce) |
| Pod token implementation | [`0x1c0b982e084451e0b272cddcce902b04c679317d`](https://testnet.snowscan.xyz/address/0x1c0b982e084451e0b272cddcce902b04c679317d) |

### Privacy Portal tokens

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| p.MTT | `0x328e70e1c52662cd5f19f824fcb8b463d77a6686` | `0x758a8F9a216A95773DDf6F73004B85d59f224518` | `0x02f284a1968160E1d3e4bC2BA3261be49725E765` |
| p.USDC | `0x5425890298aed601595a70AB815c96711a31Bc65` | `0xE75373ADb4AD1A5634a10f4644822943830b18c5` | `0x21576D8CCE47d044C5815bd59eca1F6DA94c65A5` |
| p.WAVAX | `0xd00ae08403B9bbb9124bB305C09058E32C39A48c` | `0xe6932f6Ab846bf389f7ef355dd5830594623B8E5` | `0x74d47cD68203066c97BA99787Fe1e0c68Ce42b04` |

## How this network fits PoD

| Piece | Role on Fuji |
| --- | --- |
| Inbox | Host-side courier: accepts encrypted requests from your dApp and delivers COTI callbacks |
| Your dApp | Configures Inbox + COTI chain ID `7082400` + [MPC executor on COTI Testnet](coti-testnet.md#pod-contracts) |
| Price oracle | Converts Fuji fee budgets against COTI-side costs |

Flow at a glance: **user / dApp on Fuji → Fuji Inbox → (relayer) → COTI Inbox → MPC executor → callback to Fuji**.

## SDK constants

In `@coti-io/pod-sdk` / `@coti-io/coti-contracts`:

| Constant | Value |
| --- | --- |
| Chain ID | `43113` |
| Inbox | `0x3b8B70819f27e0438cBcE7f31894f799da52648F` (`FUJI_DEFAULT_INBOX_ADDRESS`) |

Point `configureCoti` at the [COTI Testnet MPC executor](coti-testnet.md#pod-contracts) and chain ID `7082400`.
