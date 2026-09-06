# What is Privacy on Avalanche?

## The problem PoD addresses

Public blockchains are excellent for **agreement**, **auditability**, and **open participation**. They are weaker when an application must hide **amounts**, **scores**, **personal fields**, or **business logic inputs** from everyone who can read the chain.

Classic options include “only encrypt off-chain” (which weakens composability) or “move everything to a specialized chain” (which may not fit your custody, liquidity, or regulatory story).

**Privacy on Avalanche** applies the **Privacy on Demand** middle path to **Avalanche Fuji C-Chain**:

- Keep **accounts, tokens, and workflow** on **Avalanche** (Fuji for test; the same pattern targets Avalanche C-Chain style EVM hosts).
- Send **encrypted inputs** into a **private computation** environment on **COTI**.
- Receive **encrypted outputs** back on Fuji, which **only authorized users** can decrypt on their own devices.

So: **Avalanche for coordination**, **COTI for the sensitive math and data handling**.

## What “privacy” means in this model

PoD is not one magic switch. In practice, privacy here means:

1. **Sensitive values are not stored on Fuji in plaintext** in the way ordinary `uint256` balances are.
2. **Computation over those values** happens in COTI’s private execution path, not in a trace everyone can replay from Fuji calldata.
3. **Users decrypt only what they are entitled to see**, using **client-side keys** (for example an account AES key managed through your wallet or onboarding flow), as described in the SDK’s client integration guides.

What PoD does **not** automatically guarantee by itself:

- Perfect **metadata privacy** (timing, gas, which Fuji contract was called, and linkage patterns may still leak information unless you design around them).
- **Legal or regulatory** classification of your use case; you still need product and counsel review.

## What ships in the SDK versus what your team builds

The **COTI PoD stack** provides the pattern: TypeScript helpers in [`@coti-io/pod-sdk`](https://www.npmjs.com/package/@coti-io/pod-sdk) ([GitHub](https://github.com/coti-io/coti-sdk-pod)), and Solidity (`PodLib`, **`PodUserFuji`**, …) in [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts). Your project still typically supplies:

- **Application-specific** Fuji contracts and state machines.
- **User experience** for onboarding, showing **pending / completed / failed** private operations, and **safe key handling**.
- **Operations**: monitoring, indexing, or internal tools for stuck requests and AVAX fee configuration, as appropriate for your deployment.

The SDK’s own [documentation README](https://github.com/cotitech-io/coti-pod-sdk/blob/main/docs/README.md) states scope clearly: it does not replace deployment scripts, indexers, or backend services for you.

## Next steps

- For **Fuji network parameters and addresses**, read [Avalanche Fuji](networks/fuji.md).
- For a **walkthrough of one request**, read [How a private request travels end to end](how-a-private-request-travels-end-to-end.md).
- For **boxes-and-arrows** views of Inbox, executor, PodUser, and PodLib, read [Architecture and main components](architecture-and-components.md).
