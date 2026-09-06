# Host-chain deployment

Privacy on Demand puts the **confidential wrapper on the host chain**. Encrypted computation runs on COTI. The host does not need FHE precompiles, a forked EVM, or an ERC-7984 stack.

| | |
| :--- | :--- |
| Host requirement | An EVM chain with a PoD Inbox |
| Live testnets (6 Sep 2026) | Avalanche Fuji, Ethereum Sepolia |
| Token contract | Ordinary host-chain bytecode (portal + pToken clone) |
| Private execution | COTI MPC / garbled-circuit path, via Inbox request/callback |

Zama FHEVM confidential tokens instead assume FHEVM host contracts, Relayer/Gateway, and coprocessors on that host.

Inbox and executor roles: [Architecture and main components](../privacy-on-demand/architecture-and-components.md). Live addresses: [Deployed contracts](deployed-contracts.md). ERC-7984 relationship: [Compatibility and divergence](compatibility-and-divergence.md).
