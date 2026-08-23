# Ethereum Sepolia

Ethereum Sepolia is a supported **host chain** for Privacy on Demand. Your dApp contracts and the local **Inbox** live here; private computation still runs on [COTI Testnet](coti-testnet.md).

> **Note:** Addresses below match `pod-ecosystem-integration/deployConfig.json` (Inbox salt `pod.inbox.v2.2`). They may change after contract redeploys. Confirm against that file or your SDK release before production use.

## Network details

| Parameter | Value |
| --- | --- |
| Network name | Ethereum Sepolia |
| Chain ID | `11155111` |
| Currency | ETH |
| RPC URL | `https://rpc.sepolia.org` (or your preferred Sepolia provider) |
| Block explorer | [https://sepolia.etherscan.io](https://sepolia.etherscan.io) |

Private execution for Sepolia dApps targets **COTI Testnet** (`7082400`). See [COTI Testnet](coti-testnet.md) for the MPC executor and COTI-side Inbox.

## PoD contracts

| Contract | Address | Description |
| --- | --- | --- |
| Inbox | [`0x3b8B70819f27e0438cBcE7f31894f799da52648F`](https://sepolia.etherscan.io/address/0x3b8B70819f27e0438cBcE7f31894f799da52648F) | Cross-chain message router (CREATE3; same address on every PoD chain) |
| Price oracle | [`0x71f0deac8adb89b7f1b09b38e2531e06bcca0b03`](https://sepolia.etherscan.io/address/0x71f0deac8adb89b7f1b09b38e2531e06bcca0b03) | Local/remote token prices used by Inbox fee conversion |
| MpcAdder (example) | [`0xacc74e07db35fb740509fc82228e631f4f08441c`](https://sepolia.etherscan.io/address/0xacc74e07db35fb740509fc82228e631f4f08441c) | Reference primitive-only adder dApp on Sepolia |

## PoD cross-chain Privacy Portal (Sepolia)

This is the **PoD host-chain Privacy Portal** (factory + pTokens on Sepolia, private compute via COTI Inbox). It is **not** the [native COTI Privacy Portal / PrivateERC20](../../coti-privacy-portal/README.md) that runs entirely on COTI.

| Contract | Address |
| --- | --- |
| Privacy Portal factory | [`0x11a27bdf2b2c251609d78d5c9b53b3c2d71d663c`](https://sepolia.etherscan.io/address/0x11a27bdf2b2c251609d78d5c9b53b3c2d71d663c) |
| Portal implementation | [`0x9f4607850ebff3e364bea998816b13f7a59933fb`](https://sepolia.etherscan.io/address/0x9f4607850ebff3e364bea998816b13f7a59933fb) |
| Pod token implementation | [`0x3d6510ad5b4139f4fc2f8e94ab9d740e1b9f72ad`](https://sepolia.etherscan.io/address/0x3d6510ad5b4139f4fc2f8e94ab9d740e1b9f72ad) |

### Privacy Portal tokens

| Token | Underlying | Portal | pToken |
| --- | --- | --- | --- |
| p.MTT | `0xd3f5c63f4D87D2235b295FbA83351d31d0eD1BeE` | `0x7e1fecDBC7393A7165Ae7f5F1c56baA4D12c6fc0` | `0x0510F0b32828D5fB472dE5A5bE30b370c5D1a056` |
| p.USDC | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` | `0x0D02bD729698630c6f9776cDE1C8e5E00146202e` | `0xD7B3D49F85000489708B7db5B0f1a8693Fc707f3` |
| p.WETH | `0x7b79995e5f793A07Bc00c21412e50Ecae098E7f9` | `0x2C8B9bBeC8604143863c90D02FdF626a7D5a51C5` | `0xd33A363459c6Ee0C4F8504E380E8D3Aa4F209116` |

## How this network fits PoD

| Piece | Role on Sepolia |
| --- | --- |
| Inbox | Host-side courier: accepts encrypted requests from your dApp and delivers COTI callbacks |
| Your dApp | Configures Inbox + COTI chain ID `7082400` + [MPC executor on COTI Testnet](coti-testnet.md#pod-contracts) |
| Price oracle | Converts Sepolia fee budgets against COTI-side costs |

Flow at a glance: **user / dApp on Sepolia → Sepolia Inbox → (relayer) → COTI Inbox → MPC executor → callback to Sepolia**.

Hands-on next step: [Tutorial: private Adder on Sepolia](../tutorial-private-adder-sepolia.md).

## SDK constants

In `@coti-io/pod-sdk` / `@coti-io/coti-contracts`:

| Constant | Value |
| --- | --- |
| Chain ID | `11155111` |
| Inbox | `0x3b8B70819f27e0438cBcE7f31894f799da52648F` (`SEPOLIA_DEFAULT_INBOX_ADDRESS`) |
| Solidity preset | `PodUserSepolia` — sets Sepolia Inbox and configures COTI Testnet MPC executor |

Point `configureCoti` at the [COTI Testnet MPC executor](coti-testnet.md#pod-contracts) and chain ID `7082400` if you are not using `PodUserSepolia`.
