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

### System errors vs application `raise`

Both are delivered to the **same** source `errorSelector(bytes data)` (same path as `inbox.raise`). Branch with `inbox.inboxErrorType()` (`SystemError` vs `Exception`).

| Kind | When | COTI target ran? | `data` layout | Retryable via `retryFailedRequest`? |
| --- | --- | --- | --- | --- |
| **System error** | Encode / `validateCiphertext` fails before the COTI app runs | No | Inbox `{ErrorData}`: `abi.encode(uint64 code, bytes message)` (code `2`). Sender is `SYSTEM_SENDER` | **No** |
| **App `raise`** | COTI app calls `inbox.raise(...)` | Yes (started) | **dApp-defined** (e.g. `abi.encode(from, to, reason)`) | Submit a **new** request after pending clears |
| **Execution failure** | Target reverts without `raise` (code `1`) | Yes | No automatic source callback; error stored on COTI Inbox | **Yes** on COTI (permissionless) |

**Handler pattern** (see PodERC20 error callbacks):

1. `onlyInbox`; `_errorCallbackContext()` **reverts** unless `inboxErrorType()` is `SystemError`/`Exception`, `sourceRequestId` is linked, and status is Pending.
2. Branch on type: `SystemError` → decode Inbox `{ErrorData}`; `Exception` → decode your app `raise` layout.

### One-way vs two-way error handling

- **`sendOneWayMessage` rejects a non-zero `errorSelector`.** One-way jobs have no return / error callback leg. If you need an `errorSelector` handler, use a **two-way** message.
- System-error and app-`raise` callbacks therefore apply to **two-way** flows that registered an `errorSelector`.

### Execution failure, capped returndata, and `getOutboxError`

When the COTI target reverts without `raise`, the miner records **error code `1`** (`ERROR_CODE_EXECUTION_FAILED`) and stores the first ≤**256** bytes of returndata in `errors[requestId].errorMessage`.

- **`getOutboxError(requestId)`** returns `(code, data)` where `data` is those same raw bytes. Decode `Error(string)` / custom errors in your client (JS/TS).
- If `data.length == 256`, the original returndata may have been longer (cap truncated it).

Anyone may call permissionless **`retryFailedRequest(requestId)`** on COTI while the stored code is still `1`. A retry that fails to **re-encode** the call **reverts** and **keeps** code `1` (it does not flip the request to encode-failed / code `2`).

### What `executed` and response events mean

Inbox flags such as **`executed`** on an incoming request, and compact events such as **`IncomingResponseReceived`**, mean the **return / error leg was ingested** by the Inbox—not that your application callback **committed** successfully.

- A return leg can still leave a **retryable** execution error (`errors[id]` with code `1`) if the callback reverted.
- Product and indexers should treat **application events / request status** (for example pToken `requests(id).status`) as the source of truth for user-visible success or failure—not Inbox `executed` alone.

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
