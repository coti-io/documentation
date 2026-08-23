# Reference: examples and contracts

Shipped reference contracts live in [**`@coti-io/coti-contracts`**](https://github.com/coti-io/coti-contracts/tree/main/contracts/pod/examples). They teach integration patterns — not production templates. For hardening guidance, see [Contract patterns checklist](contract-patterns-checklist.md).

## Example matrix

| Example | Path | What it demonstrates |
| --- | --- | --- |
| MPC Adder | [`MpcAdder.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/MpcAdder.sol) | Minimal `PodLib` async request and `ctUint64` callback storage |
| Pod Adder 128 | [`PodAdder128.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/it128/PodAdder128.sol) | 128-bit lane primitive flow |
| Pod Adder 256 | [`PodAdder256.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/it256/PodAdder256.sol) | 256-bit lane primitive flow |
| Pausable Adder | [`MpcAdderPausable.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/examples/MpcAdderPausable.sol) | Same as `MpcAdder`, but callback reverts while paused — useful for testing fault paths |

## Related token and portal contracts

For private token flows beyond primitives:

| Area | Starting point |
| --- | --- |
| Private ERC20 (PoD) | [`PodERC20.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/token/perc20/PodERC20.sol) and [`token/perc20/`](https://github.com/coti-io/coti-contracts/tree/main/contracts/pod/token/perc20) |
| ERC-7984 portal tokens | [`token/erc7984/`](https://github.com/coti-io/coti-contracts/tree/main/contracts/pod/token/erc7984) |
| Privacy portal | [`privacy/PrivacyPortal.sol`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/privacy/PrivacyPortal.sol) |

Live demos: [pod.coti.io](https://pod.coti.io/) (MpcAdder journey), [millionaire.demo.coti.io](https://millionaire.demo.coti.io) (comparison game — source may differ from current repo layout).

## `MpcAdder.sol`

Key ideas:

- Extend `PodLib` with a network preset (`PodUserSepolia` or similar).
- Accept `itUint64` inputs on a payable entrypoint.
- Pass `msg.value` and `callbackFeeLocalWei` into `add64`.
- Decode callback payload as `ctUint64`.

**Start here** for the smallest working async private flow. Step-by-step guide: [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md). Production hardening: [Contract patterns checklist](contract-patterns-checklist.md).

## `PodAdder128.sol` / `PodAdder256.sol`

Same pattern as `MpcAdder` at wider bit widths. Use when your product needs `itUint128` / `itUint256` lanes and matching `PodLib128` / `PodLib256` helpers.

## `MpcAdderPausable.sol`

Extends `MpcAdder` with OpenZeppelin `Pausable`. The success callback (`receiveC`) is gated with `whenNotPaused`, so **private execution on COTI can still complete** while the **callback on your host chain reverts** if the contract is paused.

Use this example to test fault scenarios end to end:

1. Submit an `add` request while the contract is **paused**.
2. Observe the outbound leg succeed on COTI, but the callback fail when the Inbox invokes `receiveC`.
3. Verify your client handles the failure — for example [`PodRequest`](typescript-pod-sdk.md) reporting `response.execution`, `onDefaultMpcError` / `ErrorRemoteCall` events, or UI **failed** state ([Async private operations](async-private-operations.md)).
4. Call `unpause()` and retry, or confirm stuck requests behave as your product expects.

This is primarily a **testing and integration** reference, not a production pause design. For operational controls in production, pair pause logic with explicit request status, error callbacks, and support runbooks ([Contract patterns checklist](contract-patterns-checklist.md)).

## Custom-logic patterns (not in `examples/`)

Stateful comparison games and custom COTI orchestration follow the split-contract model in:

- [Tutorial: custom privacy logic with PoD](tutorial-custom-logic.md)
- [Cookbook: private investor allocations](cookbook-private-investor-allocations.md)

## Suggested study order

1. `MpcAdder.sol` + [private Adder tutorial](tutorial-private-adder-sepolia.md)
2. `MpcAdderPausable.sol` when you need to test callback failure (e.g. paused contract)
3. `PodAdder256.sol` if you need 256-bit lanes
4. Custom logic tutorial when primitives are insufficient
5. Token/portal contracts when building private assets

## See also

- [Reference: PodLib primitives](reference-podlib-and-primitives.md)
- [Tutorials: building PoD dApps](tutorials-privacy-on-demand.md)
