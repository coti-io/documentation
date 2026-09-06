# Networks

Privacy on Demand spans **two domains**:

1. **Host chain** — where your dApp contracts, assets, and the local **Inbox** live (for example Avalanche Fuji).
2. **COTI** — where private computation runs via the **MPC executor** and the COTI-side **Inbox**.

The pages below list network parameters and deployed contract addresses for current test environments. Addresses can change after redeploys; treat the [PoD SDK](https://github.com/cotitech-io/coti-pod-sdk) and your environment config as the live source of truth when building against a specific release.

| Network | Chain ID | Role in PoD |
| --- | --- | --- |
| [COTI Testnet](coti-testnet.md) | `7082400` | Private execution (MPC executor, COTI Inbox) |
| [Avalanche Fuji](fuji.md) | `43113` | Host chain (dApp + Inbox paired with COTI Testnet) |

The shared CREATE3 **Inbox** address is the same on every supported chain:

`0x3b8B70819f27e0438cBcE7f31894f799da52648F`

For general COTI chain RPC and faucet details (not PoD-specific), see the top-level [Networks](../../networks/README.md) section.
