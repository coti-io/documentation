# What is Privacy on Demand?

## The problem PoD addresses

Public blockchains are excellent for **agreement**, **auditability**, and **open participation**. They are weaker when an application must hide **amounts**, **scores**, **personal fields**, or **business logic inputs** from everyone who can read the chain.

Classic options include “only encrypt off-chain” (which weakens composability) or “move everything to a specialized chain” (which may not fit your custody, liquidity, or regulatory story).

**Privacy on Demand** offers a middle path:

- Keep **accounts, tokens, and workflow** where you already run them: on an **EVM-compatible chain**.
- Send **encrypted inputs** into a **private computation** environment on **COTI**.
- Receive **encrypted outputs** back on your chain, which **only authorized users** can decrypt on their own devices.

So: **public chain for coordination**, **COTI for the sensitive math and data handling**.

## What “privacy” means in this model

PoD is not one magic switch. In practice, privacy here means:

1. **Sensitive values are not stored on the orchestrating chain in plaintext** in the way ordinary `uint256` balances are.
2. **Computation over those values** happens in COTI’s private execution path, not in a trace everyone can replay from calldata.
3. **Users decrypt only what they are entitled to see**, using **client-side keys** (for example an account AES key managed through your wallet or onboarding flow), as described in the SDK’s client integration guides.

What PoD does **not** automatically guarantee by itself:

- Perfect **metadata privacy** (timing, gas, which contract was called, and linkage patterns may still leak information unless you design around them).
- **Legal or regulatory** classification of your use case; you still need product and counsel review.

## What ships in the SDK versus what your team builds

The **COTI PoD stack** provides the pattern: TypeScript helpers in [`@coti-io/pod-sdk`](https://www.npmjs.com/package/@coti-io/pod-sdk) ([GitHub](https://github.com/coti-io/coti-sdk-pod)), and Solidity (`PodLib`, **`PodUserSepolia`**, …) in [`@coti-io/coti-contracts`](https://github.com/coti-io/coti-contracts). Your project still typically supplies:

- **Application-specific** EVM contracts and state machines.
- **User experience** for onboarding, showing **pending / completed / failed** private operations, and **safe key handling**.
- **Operations**: monitoring, indexing, or internal tools for stuck requests and fee configuration, as appropriate for your deployment.

Those packages do not replace your deployment scripts, indexers, or backend services.

## Next steps

- For a **walkthrough of one request**, read [How a private request travels end to end](how-a-private-request-travels-end-to-end.md).
- For **boxes-and-arrows** views of Inbox, executor, PodUser, and PodLib, read [Architecture and main components](architecture-and-components.md).
