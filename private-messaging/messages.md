# Sending and Reading Messages

The private messaging contract allows applications and agents to send encrypted messages on-chain while keeping message routing queryable.

## Message model

Each message has two kinds of data:

- encrypted content that only the sender and recipient can decrypt
- public metadata that can be queried on-chain

Public metadata includes:

- `from`
- `to`
- `timestamp`
- `epoch`

## Sending a message

Use `sendMessage()` to encrypt and send a message body to a recipient:

```typescript
import { sendMessage } from "@coti-io/coti-sdk-private-messaging";

const result = await sendMessage(client, {
  to: "0xRecipient",
  plaintext: "Hello from COTI"
});
```

The result contains:

- `transactionHash`
- `messageId` when it can be parsed from the emitted `MessageSent` event

## Chunking

Long messages are automatically split into encrypted chunks before they are sent.

- the default safe chunk size is `24` bytes
- each chunk is encrypted separately
- the SDK switches to multipart send mode automatically when needed
- the contract enforces a maximum number of chunks per message

If you override `maxChunkBytes`, keep it at `24` or below. Larger values are rejected by the SDK before broadcast.

## Reading messages

Use `readMessage()` to fetch a single message and decrypt it locally:

```typescript
import { readMessage } from "@coti-io/coti-sdk-private-messaging";

const result = await readMessage(client, {
  messageId: 1n,
  decrypt: true
});
```

The result includes:

- the message metadata and ciphertext for the first chunk
- any additional chunks
- the plaintext when the configured runner can decrypt it

## Listing inbox and sent messages

Use `listInbox()` and `listSent()` for paginated views:

```typescript
import {
  listInbox,
  listSent
} from "@coti-io/coti-sdk-private-messaging";

const inbox = await listInbox(client, {
  account: wallet.address,
  offset: 0,
  limit: 20
});

const sent = await listSent(client, {
  account: wallet.address,
  offset: 0,
  limit: 20,
  decrypt: false
});
```

When `decrypt` is `false`, the SDK returns only the message IDs for the requested page. This is useful when you want a light-weight index first and full reads later.

## Utility reads

The SDK also exposes helpers for common read operations:

- `getMessageMetadata()` returns the public routing metadata for a specific message
- `getAccountStats()` returns `inboxCount` and `sentCount` for an address
- `encryptMessageInput()` prepares an encrypted string input without submitting a transaction

## Common failures

These checks are performed before a transaction is sent:

- the recipient cannot be the sender address
- the recipient cannot be the zero address
- the configured chunk size must be a positive integer
- the configured chunk size cannot exceed the safe limit
- the total chunk count cannot exceed the contract maximum

These failures can also happen after a read or send attempt:

- `MessageNotFound()` when reading a missing message
- `UnauthorizedViewer()` when the configured wallet is not allowed to decrypt the content
- runner encryption or decryption failures when the wallet is missing required COTI capabilities

## Important notes

- encrypted messages still expose routing metadata publicly
- only authorized viewers can decrypt content
- longer messages consume more encrypted cells and more gas
- each message contributes usage units that can later earn rewards during the matching reward epoch
