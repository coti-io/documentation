# Networks (Privacy on Avalanche)

Privacy on Avalanche spans **two domains**:

1. **Avalanche Fuji C-Chain** — where your dApp contracts, assets (paid in **AVAX**), and the local **Inbox** live.
2. **COTI Testnet** — where private computation runs via the **MPC executor** and the COTI-side **Inbox**.

The pages below list network parameters and deployed contract addresses for the current Fuji ↔ COTI Testnet pairing. Addresses can change after redeploys; treat **`pod-ecosystem-integration` deployConfig** ([`deployConfig.json`](https://github.com/coti-io/pod-ecosystem-integration) — `inboxSalt` + `chains.<id>.inbox`) as the operator source of truth, and [`@coti-io/pod-sdk`](https://github.com/coti-io/coti-sdk-pod) / [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts) and your environment config when building against a specific release.

| Network | Chain ID | Role |
| --- | --- | --- |
| [Avalanche Fuji](fuji.md) | `43113` | **Host chain** — dApp + Inbox (native token: AVAX) |
| [COTI Testnet](coti-testnet.md) | `7082400` | **Private execution** — MPC executor + COTI Inbox |

The shared CREATE3 **Inbox** address is the same on every supported PoD chain (from deployConfig `inboxSalt` family `pod.inbox.v2.2`):

`0x3b8B70819f27e0438cBcE7f31894f799da52648F`

**Two Privacy Portal worlds**

| Portal | Where | Docs |
| --- | --- | --- |
| **PoD cross-chain Privacy Portal** | Fuji factory + host pTokens; compute via COTI Inbox | [Avalanche Fuji](fuji.md) |
| **Native COTI Privacy Portal** | PrivateERC20 / bridges on COTI only | [COTI Privacy Portal](../../coti-privacy-portal/README.md) |

For Sepolia host-chain addresses (same PoD pattern), see [Ethereum Sepolia](../../privacy-on-demand/networks/sepolia.md) in the Privacy on Demand book.

**Quick links for Fuji builders**

- RPC: `https://api.avax-test.network/ext/bc/C/rpc`
- Explorer: [testnet.snowscan.xyz](https://testnet.snowscan.xyz)
- Solidity preset: `PodUserFuji`
- SDK constant: `FUJI_DEFAULT_INBOX_ADDRESS`

Start here if you need wallet + faucet + Hardhat/Foundry first: **[Getting started on Avalanche Fuji (Day 0)](../getting-started-fuji.md)**.

For general COTI chain RPC and faucet details (not PoD-specific), see the top-level [Networks](../../networks/README.md) section.
