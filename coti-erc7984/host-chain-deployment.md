# Host-chain deployment

## Deploy anywhere there is an Inbox

Confidential tokens have historically meant moving to a privacy chain and asking your users to follow. Privacy on Demand inverts that: the **confidential wrapper lives on your chain**, and the encrypted computation happens on COTI behind the scenes.

This is host-chain deployment of COTI confidential tokens — not a requirement that the host run FHE precompiles or adopt ERC-7984.

|                                |                                                                 |
| :----------------------------- | :-------------------------------------------------------------- |
| **Chains supported**           | Any EVM chain with a PoD Inbox deployed                         |
| **Live today**                 | Avalanche Fuji, Ethereum Sepolia                                |
| **Required of the host chain** | Nothing — no FHE precompiles, no custom opcodes, no forked EVM |
| **Required of the user**       | A standard wallet                                               |

Your liquidity, your users, and your existing integrations stay exactly where they are.

For the components behind the Inbox, see [Architecture and main components](../privacy-on-demand/architecture-and-components.md) in the Privacy on Demand section. For how COTI relates to ERC-7984, see [Compatibility and divergence](compatibility-and-divergence.md).
