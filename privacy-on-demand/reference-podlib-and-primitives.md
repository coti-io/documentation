# Reference: PodLib primitives

Source: [`PodLib.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib.sol) and bit-width modules [`PodLib64.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib64.sol), [`PodLib128.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib128.sol), [`PodLib256.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodLib256.sol) in **`@coti-io/coti-contracts`**.

`PodLib` is the fast path for **built-in private operations** on COTI without writing a custom COTI contract. It inherits `PodLibBase`, which forwards total and callback native fees to `IInbox.sendTwoWayMessage`.

## Network presets

Compose `PodLib` with a chain preset mixin:

| Mixin | Host chain | Source |
| --- | --- | --- |
| `PodUserSepolia` | Ethereum Sepolia (11155111) | [`PodUserSepolia.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserSepolia.sol) |
| `PodUserFuji` | Avalanche Fuji (43113) | [`PodUserFuji.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserFuji.sol) |

Presets call `setInbox` and `configureCoti` in their constructor. Your contract only needs:

```solidity
contract MyApp is PodLib, PodUserSepolia {
    constructor() PodLibBase(msg.sender) {}
}
```

Network constants (inbox address, COTI chain id, MPC executor) live in [`PodNetworkConstants.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/PodNetworkConstants.sol). Confirm values against your installed package version.

## What PodLib does

- Builds method payloads with `MpcAbiCodec`
- Routes messages to the configured COTI MPC executor
- Sends two-way Inbox requests with callback and error selectors
- Provides default error handler `onDefaultMpcError(bytes32 requestId)`

**Type mapping:** EVM helpers accept `it*` arguments; COTI executor methods use `gt*` parameters. The inbox pipeline validates and re-encodes before COTI invocation.

## Primitive families (by bit width)

Each helper follows a fee-aware shape:

```solidity
function add64(
    itUint64 memory a,
    itUint64 memory b,
    address cOwner,
    bytes4 callbackSelector,
    bytes4 errorSelector,
    uint256 totalValueWei,
    uint256 callbackFeeLocalWei
) internal returns (bytes32);
```

256-bit examples from `PodLib256` include:

| Category | Functions |
| --- | --- |
| Arithmetic | `add256`, `sub256`, `mul256`, `mulWrapping256` |
| Bitwise | `and256`, `or256`, `xor256` |
| Compare | `eq256`, `ne256`, `ge256`, `gt256`, `le256`, `lt256`, `min256`, `max256` |
| Select | `mux256` |
| Shift | `shl256`, `shr256` |
| Random | `rand256`, `randBoundedBits256` |

64- and 128-bit variants mirror the same families on `PodLib64` / `PodLib128`. See the source files for the complete matrix and exact signatures.

## Fees

Library sends go through `_sendTwoWayWithFee`:

- **`totalValueWei`** — forwarded as `msg.value` to the Inbox (entire message budget)
- **`callbackFeeLocalWei`** — slice reserved for the callback leg

See [How do PoA fees work?](how-poa-fees-work.md) for oracle conversion and worked examples.

## When to move beyond PodLib

Use custom EVM + COTI contracts when:

- The operation is not a built-in primitive
- The callback payload is a custom tuple
- COTI logic needs multiple private steps and persistent state

See [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md) and [Tutorials: building PoD dApps](tutorials-privacy-on-demand.md).

## See also

- [Reference: data types](reference-data-types.md)
- [Contract patterns checklist](contract-patterns-checklist.md)
- [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) — full walkthrough with TypeScript
