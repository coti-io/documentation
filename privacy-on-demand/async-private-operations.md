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

The SDK’s [Async execution](https://github.com/coti-io/coti-sdk-pod/tree/main/site/05a-async-execution) page lists the canonical lifecycle and common mistakes (wrong decode shape, missing `onlyInbox`, expecting same-block completion).

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
| **Errors** | Show a clear **Failed** state with a next step. If the destination ran out of gas, tell the user to **retry that same request** in the PoD explorer — not to submit the action again. See below. |
| **Support** | Train staff that **“stuck pending”** may be **fee**, **routing**, or **downstream execution** issues—not always “user error.” |

## If the destination ran out of gas

Sometimes COTI **already finished** the private work, but writing the result back on your chain (Sepolia, Fuji, and so on) **runs out of gas**. In the app this looks like **Failed**. The encrypted result was never stored.

**Do not tap the same button again** in the dApp. That starts a **new** job. The failed one stays failed until it expires (about **48 hours**).

Instead, **retry the same request** in the PoD explorer. Anyone can do this. You pay a little native token on the **destination** network (AVAX on Fuji, ETH on Sepolia, COTI on COTI). That retry is allowed extra gas, so the callback can finish.

Retry helps when the explorer says **Destination ran out of gas** or **Prepaid gas was used up**. It does **not** help if the destination contract itself rejected the call (a named error), if the miner never accepted the message, if the retry window has closed, or if the page already says **Recovered**.

### How to retry in the explorer

1. Open the testnet explorer: [https://testnet.explorer.pod.coti.io](https://testnet.explorer.pod.coti.io).
2. Find your request: click **Failed**, or press **`/`**, paste the **request ID**, and press Enter.
3. Open the message.
4. Read the banner at the top. If it is an out-of-gas failure, you will see **Retry on** the destination chain.
5. Keep the suggested **gas limit** unless support told you to raise it. Confirm in your wallet on that same network (the page can switch the wallet for you). You pay that chain’s token, not a second dApp fee.
6. Wait until the transaction confirms. The page should show **Recovered**. You can then decrypt the result as usual. The original failure stays in the event history on purpose.

If the wallet or the page says the retry failed:

| What you see | What to do |
| --- | --- |
| Already recovered / not a failed request | Refresh the page. Nothing more to send. |
| Still failed / execution failed | Raise the gas limit and try once more, or the destination app is still rejecting the call. |
| Encode failed | Retry will not fix this. Contact support with the request ID. |
| Window closed / expired | Too late to retry this request. You would need a new job from the dApp. |

{% hint style="info" %}
The explorer **simulates** the retry before opening the wallet. If simulation fails but the page offers **Send anyway**, that can happen on COTI even when a real retry would work. Use it only in that case.
{% endhint %}

## Relationship to decryption

Even after **completion**, plaintext is **not** magically public on-chain. **Authorized clients** decrypt **`ct*` outputs** locally. Planning must cover **key recovery**, **device loss**, and **clear disclosure** of who can decrypt what.

## Next steps

- [How a private request travels end to end](how-a-private-request-travels-end-to-end.md) — full path diagram.
- [For developers: mapping concepts to the SDK](for-developers-mapping-to-the-sdk.md) — testing and callback checklists.
