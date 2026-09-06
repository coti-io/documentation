# Concurrency

How overlapping confidential transfers are ordered.

## COTI PoD (host-chain pTokens)

A user can submit **multiple** Inbox requests while earlier ones are still in flight. Each request has a **request id**. A **monotonic nonce** on the COTI-side token applies results in order so two in-flight transfers from the same account do not clobber each other.

The host-chain mental model is still **async**: submit → wait → callback. See [Async private operations](../privacy-on-demand/async-private-operations.md).

## Zama / FHEVM ERC-7984-style tokens

Host-chain transactions follow the ordinary **EVM account nonce**. One account cannot have two pending host txs mined out of nonce order.

Coprocessors execute FHE ops in the order of host events for a given handle. There is no PoD-style in-flight **request queue** on the token: a `confidentialTransfer` that already passed Relayer input registration is a single host transaction.

Flows that need a **public** result take **two host transactions** (for example `unwrap` then `finalizeUnwrap`). Those two txs are sequenced by the EVM nonce and by waiting for KMS signatures in between — not by a PoD request id.

## Transfer semantics

Silent insufficient-balance handling (encrypted amount → effective zero) is independent of this queueing model. See [Transfer semantics](transfer-semantics.md).
