# Avalanche Fuji

Avalanche Fuji is a supported **host chain** for Privacy on Demand. Your dApp contracts and the local **Inbox** live here; private computation still runs on [COTI Testnet](coti-testnet.md).

> **Note:** Addresses below reflect the current PoD test deployment. They may change after contract redeploys. Confirm against your SDK release or environment before production use.

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
| Inbox | [`0xAb625bE229F603f6BBF964474AFf6d5487e364De`](https://testnet.snowscan.xyz/address/0xAb625bE229F603f6BBF964474AFf6d5487e364De) | Cross-chain message router (CREATE3; same address on every PoD chain) |
| Price oracle | [`0xbf615045803edb0dbb2c9b9afedcd782c383a09b`](https://testnet.snowscan.xyz/address/0xbf615045803edb0dbb2c9b9afedcd782c383a09b) | Local/remote token prices used by Inbox fee conversion |
| MpcAdder (example) | [`0xf51c789e4c60d33a2abcfd656e607724f56aaf0d`](https://testnet.snowscan.xyz/address/0xf51c789e4c60d33a2abcfd656e607724f56aaf0d) | Reference primitive-only adder dApp on Fuji |

## Privacy Portal (Fuji)

Deployed factory and implementations for private-token flows on Fuji:

| Contract | Address |
| --- | --- |
| Privacy Portal factory | [`0x5230856b5ce21ee5efdd113f8d4849a4b02d3a57`](https://testnet.snowscan.xyz/address/0x5230856b5ce21ee5efdd113f8d4849a4b02d3a57) |
| Portal implementation | [`0x01dee3b8046a94896fe97d531aa2019ea2557ce3`](https://testnet.snowscan.xyz/address/0x01dee3b8046a94896fe97d531aa2019ea2557ce3) |
| Pod token implementation | [`0x0c8ec0f93cae026db214755388d2c41c8b5be08d`](https://testnet.snowscan.xyz/address/0x0c8ec0f93cae026db214755388d2c41c8b5be08d) |

### Privacy Portal tokens

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| pMTT | `0x328e70e1c52662cd5f19f824fcb8b463d77a6686` | `0x9a82B356c9f7F59aE2c04200358122eb99aE6364` | `0xc1258E5C04A6933940105FE94cC9A4C2439d3402` |
| pUSDC | `0x5425890298aed601595a70AB815c96711a31Bc65` | `0x32146Ce8a96F2b0102c5C0f27e45E53eA30ef18B` | `0xAd40C2E98ef9d37827F5206D0a6641c29f2247c5` |
| pWAVAX | `0xd00ae08403B9bbb9124bB305C09058E32C39A48c` | `0xdf75091D4AF11b94FA318eeBC6CEBC106899E25C` | `0x44392E67f6Fc5179BD25302519b1B85B87afc5D8` |

## How this network fits PoD

| Piece | Role on Fuji |
| --- | --- |
| Inbox | Host-side courier: accepts encrypted requests from your dApp and delivers COTI callbacks |
| Your dApp | Configures Inbox + COTI chain ID `7082400` + [MPC executor on COTI Testnet](coti-testnet.md#pod-contracts) |
| Price oracle | Converts Fuji fee budgets against COTI-side costs |

Flow at a glance: **user / dApp on Fuji → Fuji Inbox → (relayer) → COTI Inbox → MPC executor → callback to Fuji**.

## SDK constants

In `@coti-io/pod-sdk`:

| Constant | Value |
| --- | --- |
| Chain ID | `43113` |
| Inbox | `0xAb625bE229F603f6BBF964474AFf6d5487e364De` (`FUJI_DEFAULT_INBOX_ADDRESS`) |

Point `configureCoti` at the [COTI Testnet MPC executor](coti-testnet.md#pod-contracts) and chain ID `7082400`.
