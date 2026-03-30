# Private Messaging

Private messaging on COTI combines encrypted on-chain messages, a TypeScript SDK, a reward system for message activity, and onboarding flows for newly created wallets.

In this section you will find:

- an overview of the [`@coti-io/coti-sdk-private-messaging`](typescript-sdk.md) package
- guides for [sending and reading messages](messages.md)
- documentation for the [reward epoch system](rewards.md)
- documentation for the [starter grant flow](starter-grant.md)
- a reference for the available [skills](skills/README.md)

## How private messaging works

The private messaging system stores encrypted message bodies on-chain while keeping routing metadata queryable.

- the message body is encrypted using COTI-compatible encryption before it is sent
- only the sender and recipient can decrypt the message content
- routing metadata such as `from`, `to`, `timestamp`, and `epoch` remains public
- long messages are automatically split into multiple encrypted chunks
- message activity contributes usage units that can later earn rewards

## What to read next

If you want to build with the SDK, start with [TypeScript SDK](typescript-sdk.md).

If you want to understand the messaging flow itself, continue with [Sending and Reading Messages](messages.md).

If you want to use the agent workflows, go to [Skills](skills/README.md).
