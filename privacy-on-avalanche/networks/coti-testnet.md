# COTI Testnet

COTI Testnet is the **private execution** side of Privacy on Demand. Host-chain dApps (for example on [Avalanche Fuji](fuji.md)) send encrypted work through the cross-chain Inbox; COTI runs the private computation and returns encrypted results.

> **Note:** Addresses below reflect the current PoD test deployment. They may change after contract redeploys. Confirm against your SDK release or environment before production use.

## Network details

| Parameter | Value |
| --- | --- |
| Network name | COTI Testnet |
| Chain ID | `7082400` |
| Currency | COTI |
| RPC URL | `https://testnet.coti.io/rpc` |
| WebSocket URL | `wss://testnet.coti.io/ws` |
| Block explorer | [https://testnet.cotiscan.io](https://testnet.cotiscan.io) |
| Status | [https://uptime.coti.io](https://uptime.coti.io) |

Also see the general COTI [TestNet](../../networks/testnet/README.md) page for MetaMask setup and the [Testnet faucet](../../networks/testnet/faucet.md).

## PoD contracts

| Contract | Address | Description |
| --- | --- | --- |
| Inbox | [`0xAb625bE229F603f6BBF964474AFf6d5487e364De`](https://testnet.cotiscan.io/address/0xAb625bE229F603f6BBF964474AFf6d5487e364De) | Cross-chain message router (CREATE3; same address on every PoD chain) |
| MPC executor | [`0x68e151b78d51cea01eef6ee354579e044606a739`](https://testnet.cotiscan.io/address/0x68e151b78d51cea01eef6ee354579e044606a739) | Entry point for library-style private operations (`configureCoti` target on host dApps) |
| Price oracle | [`0xe1afeda542d1b9003df0e133187f845dea8b1ba8`](https://testnet.cotiscan.io/address/0xe1afeda542d1b9003df0e133187f845dea8b1ba8) | Local/remote token prices used by Inbox fee conversion |
| Pod ERC20 mother | [`0x293daf267bf657b0bae870a0ce8cd59f1e5eb32a`](https://testnet.cotiscan.io/address/0x293daf267bf657b0bae870a0ce8cd59f1e5eb32a) | COTI-side registry for Privacy Portal / pToken flows |

## Protocol helpers (COTI Testnet)

These are not PoD-specific, but most PoD clients need them for account onboarding and chain tooling:

| Contract | Address |
| --- | --- |
| AccountOnboard | [`0x536A67f0cc46513E7d27a370ed1aF9FDcC7A5095`](https://testnet.cotiscan.io/address/0x536A67f0cc46513E7d27a370ed1aF9FDcC7A5095) |
| MPCInterface (precompile) | `0x0000000000000000000000000000000000000064` |

Full COTI Testnet protocol addresses: [Contracts Addresses](../../networks/testnet/contracts-addresses.md).

## How this network fits PoD

| Piece | Role on COTI Testnet |
| --- | --- |
| Inbox | Receives mined requests from host chains and routes callbacks back |
| MPC executor | Target address host dApps pass to `configureCoti(..., COTI_TESTNET_CHAIN_ID)` |
| Price oracle | Converts fee budgets between host gas units and COTI |

Typical pairing for this section:

- **Host:** [Avalanche Fuji C-Chain](fuji.md) · **Execution:** COTI Testnet (see the [private Adder tutorial](../tutorial-private-adder-fuji.md))

## SDK constants

In `@coti-io/pod-sdk` and `@coti-io/coti-contracts`:

| Constant | Value |
| --- | --- |
| Chain ID | `7082400` |
| Inbox | `0xAb625bE229F603f6BBF964474AFf6d5487e364De` (`COTI_TESTNET_DEFAULT_INBOX_ADDRESS`) |

Confirm the MPC executor against your deployed release — presets in `PodNetworkConstants` can lag a redeploy; wire the address from the table above (or your env) in `configureCoti`.
