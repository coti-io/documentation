# Tutorial: custom privacy logic with PoD

This tutorial describes the **second PoD integration model**: a **custom COTI-side contract** paired with a **host-chain (for example Sepolia) contract**. Use this path when [primitive-only `PodLib` / MpcLib operations](tutorials-privacy-on-demand.md#1-primitive-only-dapps-mpclib--podlib) are not enough for your business logic.

For the conceptual split between the two models and a comparison diagram, start at **[Tutorials: building Privacy on Demand (PoD) dApps](tutorials-privacy-on-demand.md)**.

## Why two contracts?

The **host EVM chain** is excellent for accounts, assets, and user-facing flows, but your Sepolia (or mainnet) contract **cannot** host the full garbled-circuit lifecycle: it is not the right place to keep **intermediate encrypted MPC state** (`gt*`) and arbitrarily compose it the way the **COTI** execution environment does.

So you **partition** the design:

- **Sepolia side** — Collect **encrypted user inputs** (`it*`), pay Inbox fees, and issue **remote calls** to COTI. In the success path, receive **encrypted outputs** (`ct*`) in an **`onlyInbox` callback** and store or emit them for clients. Think of this layer as a **thin, client-facing orchestrator**.
- **COTI side** — Implement the **actual private logic**: receive `gt*` / `it*` payloads from the Inbox, run `MpcCore` operations, optionally **off-board** ciphertext to a specific user (`offBoardToUser`), and **`inbox.respond`** with an ABI-encoded blob the Sepolia callback decodes.

The example below mirrors an **encrypted messaging** flow: the message is handled privately on COTI so the recipient-bound ciphertext is produced in a controlled way; Sepolia only routes the call and persists **`ctString`** for the UI.

## Security habits on the host chain

Any **`onlyInbox` callback** that mutates state must **verify the caller** is the Inbox **and** that the **source COTI contract and chain** match your deployment. Otherwise a malicious remote peer could try to impersonate your COTI logic.

Always use something equivalent to:

```solidity
(uint256 callerChain, address callerContract) = IInbox(inbox).inboxMsgSender();
require(callerChain == EXPECTED_COTI_CHAIN_ID && callerContract == myCotiPeer, "unauthorized peer");
```

Match **`EXPECTED_COTI_CHAIN_ID`** and **`myCotiPeer`** to the values you configured at deploy time (the SDK presets expose constants such as `COTI_TESTNET_CHAIN_ID` on `PodUserSepolia`).

## COTI-side contract: `DirectMessageCotiSide`

The COTI contract **inherits `InboxUser`**, so only the Inbox can enter `receiveMessage`. It:

1. Accepts the garbled string and routing metadata (`sender`, `recipient`).
2. Uses **`MpcCore.offBoardToUser`** to convert the payload into **`ctString` ciphertext that only `recipient` can decrypt**.
3. Returns that ciphertext to the host chain via **`inbox.respond`**, ABI-encoding `sender`, `recipient`, and the ciphertext so the Sepolia callback can update storage deterministically.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "../InboxUserCotiTestnet.sol";
import "@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol";

contract DirectMessageCotiSide is InboxUserCotiTestnet {
    function receiveMessage(gtString calldata message, address sender, address recipient) external onlyInbox {
        // Off-board the garbled string into a cipher state exclusively for the recipient
        ctString memory cipherForRecipient = MpcCore.offBoardToUser(message, recipient);
        inbox.respond(abi.encode(sender, recipient, cipherForRecipient));
    }
}
```

Paths like `../InboxUser.sol` assume you follow the SDK’s example layout; adjust imports to your repo.

## Sepolia-side contract: orchestration only

The Sepolia contract **inherits `PodUserSepolia`** (or your network’s `PodUser` preset), tracks the **COTI peer address**, and:

1. **`sendMessage`** — Wraps encrypted input (`itString`) and public addresses in an **`IInbox.MpcMethodCall`** built with **`MpcAbiCodec`** (see the SDK’s [Request builder and remote calls](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/contracts/03-request-builder-and-remote-calls.md)). It sends a **two-way** message so the result comes back asynchronously.
2. **`onMessageReceived`** — Decodes the tuple produced on COTI, **re-checks `inboxMsgSender()`**, and stores **`ctString`** keyed by conversation participants (or whatever your product needs).

The Solidity below is **structurally** correct; wire **`MpcAbiCodec`**’s `create` / `addArgument` / `build` steps exactly as in your installed `@coti/pod-sdk` version (argument order and `gt`/`it` interface types **must** match the COTI method signature).

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {PodUserSepolia} from "../mpc/PodUserSepolia.sol";
import {MpcAbiCodec} from "../mpccodec/MpcAbiCodec.sol";
import {IInbox} from "../IInbox.sol";

// Interface MUST match the COTI `DirectMessageCotiSide.receiveMessage` ABI
// (including `gt` vs `it` types on each parameter).
interface IDirectMessagePod {
    function receiveMessage(gtString calldata message, address sender, address recipient) external;
}

contract DirectMessageEvm is PodUserSepolia {
    using MpcAbiCodec for MpcAbiCodec.MpcMethodCallContext;

    mapping(address => mapping(address => ctString)) public encryptedMessages;
    address public myCotiPeer;

    event MessageDispatched(bytes32 indexed requestId, address indexed sender, address indexed recipient);
    event MessageReply(bytes32 indexed requestId, address indexed sender, address indexed recipient);

    constructor(address _cotiPeer) {
        myCotiPeer = _cotiPeer;
    }

    /// @notice Dispatch encrypted `itString` to COTI; fee semantics match other two-way Pod calls.
    function sendMessage(
        itString memory encryptedMessage,
        address recipient,
        uint256 callbackFeeLocalWei
    ) external payable {
        MpcAbiCodec.MpcMethodCallContext memory callCtx =
            MpcAbiCodec.create(IDirectMessagePod.receiveMessage.selector, 3);
        callCtx = callCtx.addArgument(encryptedMessage);
        callCtx = callCtx.addArgument(msg.sender);
        callCtx = callCtx.addArgument(recipient);
        IInbox.MpcMethodCall memory methodCall = callCtx.build();

        bytes32 requestId = IInbox(inbox).sendTwoWayMessage{value: msg.value}(
            COTI_TESTNET_CHAIN_ID,
            myCotiPeer,
            methodCall,
            this.onMessageReceived.selector,
            this.onDefaultMpcError.selector,
            callbackFeeLocalWei
        );
        emit MessageDispatched(requestId, msg.sender, recipient);
    }

    /// @notice COTI contract executed `inbox.respond(abi.encode(sender, recipient, cipherForRecipient))`.
    function onMessageReceived(bytes memory resultData) external onlyInbox {
        (uint256 callerChain, address callerContract) = IInbox(inbox).inboxMsgSender();
        require(callerChain == COTI_TESTNET_CHAIN_ID && callerContract == myCotiPeer, "not allowed");

        bytes32 requestId = IInbox(inbox).inboxSourceRequestId();
        (address sender, address recipient, ctString memory message) =
            abi.decode(resultData, (address, address, ctString));

        encryptedMessages[sender][recipient] = message;
        emit MessageReply(requestId, sender, recipient);
    }
}
```

### TypeScript and deployment

This chapter stops at **Solidity** to highlight the **chain split**. For **encryption**, **Inbox fee estimation**, and **client-side decryption** of `ctString`, continue with:

- [Tutorial: private Adder on Sepolia](tutorial-private-adder-sepolia.md) — includes **`PodContract`**, **`encryptAndCallMethod`**, **`estimateFee`**, and **`extractRequestIds`** so you can copy the same client pattern to **`sendMessage`** (build **`PodMethodArgument[]`** with types that match your ABI: **`itString`** for the ciphertext argument, plain types for addresses and the callback-fee slot, **`isCallBackFee: true`** on the fee parameter).
- [TypeScript PoD SDK (`CotiPodCrypto`, `PodContract`)](typescript-pod-sdk.md) — short reference for **`CotiPodCrypto`** and **`PodContract`** with links to [`coti-pod-crypto.ts`](https://github.com/cotitech-io/coti-pod-sdk/blob/main/src/coti-pod-crypto.ts) and [`pod-method-call.ts`](https://github.com/cotitech-io/coti-pod-sdk/blob/main/src/pod-method-call.ts).
- [TypeScript integration — SDK](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/06-typescript-integration-ux-development.md)
- [Writing privacy contracts on Ethereum — SDK](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/05-writing-privacy-contracts-on-ethereum.md) (custom mode)

After **`encryptAndCallMethod("sendMessage", args, feeCfg)`** (or a raw **`ethers.Contract`** send), use **`await pod.extractRequestIds(receipt.hash)`** on the same **`PodContract`** instance so your UI stores the **`requestId`** emitted in the Inbox **`MessageSent`** logs—same helper as in the adder walkthrough.

## Summary

| Piece | Responsibility |
| --- | --- |
| **`DirectMessageCotiSide`** | Private transformation (`gtString` → recipient-bound `ctString`) and **`inbox.respond`**. |
| **`DirectMessageEvm`** | Fee-bearing Inbox send, **peer verification**, and storing **`ctString`** for the app. |
| **Client** | Encrypt inputs, submit transactions, decrypt outputs using the account key flow from the SDK. |

When your feature fits this shape—**heavy private logic on COTI**, **Sepolia as router and ciphertext cache**—you are in the **custom PoD dApp** model described on the [tutorials overview](tutorials-privacy-on-demand.md#2-custom-pod-dapps-host-chain--coti-contracts).

<div style="width:100%; box-sizing:border-box; margin:2rem 0 0 0; padding:1.35rem 1rem; border:2px solid #334155; border-radius:10px; background:#f8fafc; text-align:center;">

<p style="margin:0.4rem 0; font-size:1.25rem; font-weight:700;"><a href="tutorial-private-adder-sepolia.md" style="color:#0f172a; text-decoration:none;">Tutorial: private Adder on Sepolia</a></p>

<p style="margin:0.4rem 0; font-size:1.25rem; font-weight:700;"><a href="tutorial-custom-logic.md" style="color:#0f172a; text-decoration:none;">Tutorial: custom privacy logic with PoD</a></p>

</div>
