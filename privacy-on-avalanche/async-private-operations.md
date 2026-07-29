# Async private operations

Privacy on Demand private calls are **asynchronous**. That single sentence drives most product and operations decisions.

## What “async” means for users

On a normal smart contract call, many teams think in terms of: **submit transaction → see result in the same flow**.

With PoD, the meaningful private result usually arrives in a **second step**:

1. **Request transaction** — your dApp contract submits work **through the Inbox** and receives or emits a **request identifier**.
2. **Wait** — COTI performs private computation.
3. **Callback transaction** — the Inbox invokes your contract’s **callback** with **encrypted outputs** (`ct*`).

So the user’s mental model should be closer to **“I submitted a job”** than **“I got the answer immediately.”**

## Why the platform works this way

Private execution happens **outside** your chain’s normal synchronous EVM frame. The **Inbox** pattern exists precisely to **carry a message out** and **bring a response back** through a **controlled channel**.

The SDK’s [Async execution](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05a-async-execution.md) page lists the canonical lifecycle and common mistakes (wrong decode shape, missing `onlyInbox`, expecting same-block completion).

## What product and support teams should plan for

| Topic | Recommendation |
| --- | --- |
| **UI states** | Show **Pending / Completed / Failed** (or equivalent) per request ID. |
| **Indexing** | Expect teams to use **events**, **subgraphs**, or internal indexers to connect callbacks to user actions. |
| **Errors** | Surface **structured failure** where the SDK exposes error callbacks and codes—users need actionable next steps. |
| **Support** | Train staff that **“stuck pending”** may be **fee**, **routing**, or **downstream execution** issues—not always “user error.” |

## Relationship to decryption

Even after **completion**, plaintext is **not** magically public on-chain. **Authorized clients** decrypt **`ct*` outputs** locally. Planning must cover **key recovery**, **device loss**, and **clear disclosure** of who can decrypt what.

## Next steps

- [How a private request travels end to end](how-a-private-request-travels-end-to-end.md) — full path diagram.
- [For developers: mapping concepts to the SDK](for-developers-mapping-to-the-sdk.md) — testing and callback checklists.
