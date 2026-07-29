# Avalanche Fuji C-Chain

Avalanche Fuji C-Chain is the **primary host chain** for this Privacy on Avalanche section. Your dApp contracts and the local **Inbox** live here; fees are paid in **AVAX**; private computation still runs on [COTI Testnet](coti-testnet.md).

Use the Solidity preset **`PodUserFuji`** (or configure the Inbox + COTI executor manually) so Fuji dApps route to COTI Testnet chain ID `7082400`.

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
| Inbox | [`0x3b8B70819f27e0438cBcE7f31894f799da52648F`](https://testnet.snowscan.xyz/address/0x3b8B70819f27e0438cBcE7f31894f799da52648F) | Cross-chain message router (CREATE3; same address on every PoD chain) |
| Price oracle | [`0xf2283ca93a6747c547a961c50d0393d549c57268`](https://testnet.snowscan.xyz/address/0xf2283ca93a6747c547a961c50d0393d549c57268) | Local/remote token prices used by Inbox and Privacy Portal fee conversion |
| MpcAdder (example) | [`0xf51c789e4c60d33a2abcfd656e607724f56aaf0d`](https://testnet.snowscan.xyz/address/0xf51c789e4c60d33a2abcfd656e607724f56aaf0d) | Reference primitive-only adder dApp on Fuji |

## Privacy Portal (Fuji)

Deployed factory and implementations for private-token flows on Fuji (aligned with `@coti-io/coti-wallet-plugin` / PEI `deployConfig.json`):

| Contract | Address |
| --- | --- |
| Privacy Portal factory | [`0xf3cf653e1baee7b4e4001067780dee38991b1cbd`](https://testnet.snowscan.xyz/address/0xf3cf653e1baee7b4e4001067780dee38991b1cbd) |
| Portal implementation | [`0x63e97937e42c153cdeb25e9aca9d3d0373aec0a5`](https://testnet.snowscan.xyz/address/0x63e97937e42c153cdeb25e9aca9d3d0373aec0a5) |
| Pod token implementation | [`0xa7e4838327317f4ce6cc8b5ab07a57fdba842c77`](https://testnet.snowscan.xyz/address/0xa7e4838327317f4ce6cc8b5ab07a57fdba842c77) |

### Privacy Portal tokens

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| pMTT | `0x328e70e1c52662cd5f19f824fcb8b463d77a6686` | `0xf4100d21eB4B1a66aDde58A01D1E32356F268b3F` | `0xFC6283a9000d7D5Cf8A058A04A9ED90265Af1634` |
| pUSDC | `0x5425890298aed601595a70AB815c96711a31Bc65` | `0x090D2dc8C38275939b9381Ff2aa53012Ff412E34` | `0xe2235E064a3CEB5F1765c3b095855549d3c8A8a4` |
| pWAVAX | `0xd00ae08403B9bbb9124bB305C09058E32C39A48c` | `0x20e7239cd78BDf2E8f34c52947e54fE68D7b536F` | `0x0c58954d91392794A50F610dF8c84228D63BE9D4` |

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
| Chain ID | `43113` (`AVALANCHE_FUJI_CHAIN_ID`) |
| Inbox | `0x3b8B70819f27e0438cBcE7f31894f799da52648F` (`FUJI_DEFAULT_INBOX_ADDRESS`) |
| Solidity preset | `PodUserFuji` — sets Fuji Inbox and configures COTI Testnet MPC executor |

Point `configureCoti` at the [COTI Testnet MPC executor](coti-testnet.md#pod-contracts) and chain ID `7082400` if you are not using `PodUserFuji`.

## Getting Fuji AVAX and exploring txs

Step-by-step wallet, faucet, and toolchain setup: **[Getting started on Avalanche Fuji (Day 0)](../getting-started-fuji.md)**.

- Fund a wallet with **test AVAX**, then deploy and call your dApp with enough `msg.value` for two-way Inbox fees.
- Verify contracts and transactions on [SnowScan testnet](https://testnet.snowscan.xyz).
- Hands-on next step: [Tutorial: private Adder on Avalanche Fuji](../tutorial-private-adder-fuji.md).
