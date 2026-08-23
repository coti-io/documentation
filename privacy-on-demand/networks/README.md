# Networks

Privacy on Demand spans **two domains**:

1. **Host chain** — where your dApp contracts, assets, and the local **Inbox** live (for example Avalanche Fuji or Ethereum Sepolia).
2. **COTI** — where private computation runs via the **MPC executor** and the COTI-side **Inbox**.

The pages below list network parameters and deployed contract addresses for current test environments. Addresses can change after redeploys; treat [`deployConfig.json`](https://github.com/coti-io/pod-ecosystem-integration) (or your environment config), [`@coti-io/pod-sdk`](https://github.com/coti-io/coti-sdk-pod), and [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts) as the live source of truth when building against a specific release.

| Network | Chain ID | Role in PoD |
| --- | --- | --- |
| [COTI Testnet](coti-testnet.md) | `7082400` | Private execution (MPC executor, COTI Inbox) |
| [Avalanche Fuji](fuji.md) | `43113` | Host chain (dApp + Inbox paired with COTI Testnet) |
| [Ethereum Sepolia](sepolia.md) | `11155111` | Host chain (dApp + Inbox paired with COTI Testnet) |

The shared CREATE3 **Inbox** address (`pod.inbox.v2.2`) is the same on every supported chain:

`0x3b8B70819f27e0438cBcE7f31894f799da52648F`

**Two Privacy Portal worlds**

| Portal | Where | Docs |
| --- | --- | --- |
| **PoD cross-chain Privacy Portal** | Fuji / Sepolia factory + host pTokens; compute via COTI Inbox | Network pages above |
| **Native COTI Privacy Portal** | PrivateERC20 / bridges on COTI only | [COTI Privacy Portal](../../coti-privacy-portal/README.md) |

For general COTI chain RPC and faucet details (not PoD-specific), see the top-level [Networks](../../networks/README.md) section.
