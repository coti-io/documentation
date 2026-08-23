# Contract patterns and production checklist

Use this checklist before deploying a production PoD contract. For conceptual background, see [Architecture and main components](architecture-and-components.md) and [Async private operations](async-private-operations.md).

## Recommended contract shape

1. Inherit from `PodLib` (primitive path) or `InboxUser` (custom path).
2. Add a network preset mixin such as [`PodUserSepolia`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserSepolia.sol) or [`PodUserFuji`](https://github.com/coti-io/coti-contracts/blob/main/contracts/pod/mpc/PodUserFuji.sol) — presets auto-wire inbox and COTI routing in their constructor.
3. Accept private values as `it*` on EVM entrypoints.
4. Submit via `PodLib` helpers (e.g. `add64`, `gt256`) or `IInbox.sendTwoWayMessage{value: ...}(..., callbackFeeLocalWei)`.
5. Store `requestId` correlation state.
6. Implement success and error callbacks with `onlyInbox`.
7. Decode callback data into `ct*` and persist.
8. Expose status/result readers for async UX.

Preset wiring (`PodLib` + `PodUserSepolia` / `PodUserFuji`): [Reference: PodLib primitives](reference-podlib-and-primitives.md).

## Fee and gas checklist

- Expose **payable** entrypoints when using two-way sends; forward **`msg.value`** as the total fee and pass explicit **`callbackFeeLocalWei`**.
- Call **`calculateTwoWayFeeRequiredInLocalToken`** on the deployed Inbox before production sends (see [How do PoA fees work?](how-poa-fees-work.md)).
- Test **too-low total** and **too-low callback** reverts in addition to the happy path.

## Critical security controls

- Access-control any method that changes routing or configuration.
- Gate any call path that invokes `configureCoti(...)`.
- Keep callback handlers `onlyInbox`.
- In custom COTI flows, verify `inboxMsgSender()` matches the expected peer contract.
- Avoid external calls before state updates inside callback handlers.

## ABI correctness checklist

- Selector in `MpcAbiCodec.create(...)` matches the COTI-side target function.
- `.addArgument(...)` order matches COTI parameter order.
- Callback `abi.decode(...)` tuple matches COTI `abi.encode(...)` exactly.
- Add regression tests when callback tuple schema changes.

## Async robustness checklist

- Request starts as `PENDING` and transitions to a terminal state.
- Unknown or duplicate callback handling is defined.
- Errors are observable (`requestId`, error code, message).
- Frontend can query status by request ID (see [`PodRequest`](typescript-pod-sdk.md)).

## Privacy and type checklist

- No plaintext sensitive values in EVM storage.
- No `gt*` in EVM public interfaces.
- `ct*` used for user-decryptable outputs.
- Type width consistency preserved through the full flow (see [Reference: data types](reference-data-types.md)).

## Observability checklist

- Emit request-created event with indexed `requestId`.
- Emit completion and failure events.
- Index `MessageSent` on the Inbox for off-chain correlation (compact event carries `requestId` in indexed topics).

## Testing checklist

- Success callback updates expected state.
- Non-Inbox callback call reverts.
- Error callback updates failure state.
- Request ID mapping is deterministic.
- Frontend integration decrypts returned ciphertext correctly.
- Fee sufficiency: success with estimated fees plus buffer; reverts when under-funded.

## See also

- [Reference: examples and contracts](reference-examples-and-contracts.md)
- [Contract patterns in custom logic tutorial](tutorial-custom-logic.md)
- [Cookbook: private investor allocations](cookbook-private-investor-allocations.md)
