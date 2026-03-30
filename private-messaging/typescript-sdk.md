# TypeScript SDK

The [`@coti-io/coti-sdk-private-messaging`](https://github.com/coti-io/coti-sdk-private-messaging) package provides a TypeScript interface for encrypted messaging, reward management, starter grants, and MCP-style tool invocation.

## Installation

```bash
npm install @coti-io/coti-sdk-private-messaging @coti-io/coti-ethers
```

## Create a client

```typescript
import { Wallet, JsonRpcProvider, CotiNetwork } from "@coti-io/coti-ethers";
import {
  getDefaultCotiRpcUrl,
  createPrivateMessagingClient
} from "@coti-io/coti-sdk-private-messaging";

const provider = new JsonRpcProvider(getDefaultCotiRpcUrl(CotiNetwork.Testnet));
const wallet = new Wallet(process.env.PRIVATE_KEY!, provider);
wallet.setAesKey(process.env.AES_KEY!);

const client = createPrivateMessagingClient({
  network: CotiNetwork.Testnet,
  runner: wallet
});
```

If you do not pass a `contractAddress`, the SDK uses the built-in defaults:

- testnet RPC: `https://testnet.coti.io/rpc`
- mainnet RPC: `https://mainnet.coti.io/rpc`
- testnet messaging contract: `0xa4C514225Db5B8AE6eF1548d4CE912234A7CD954`
- mainnet messaging contract: `0xe461F448cB935a14585F6f1a30F5b4C73ffF8c05`

## Messaging APIs

The SDK exports the core messaging helpers:

```typescript
createPrivateMessagingClient(config: PrivateMessagingClientConfig): PrivateMessagingClient
encryptMessageInput(client: PrivateMessagingClient, plaintext: string)
sendMessage(client: PrivateMessagingClient, request: SendMessageRequest): Promise<SendMessageResult>
readMessage(client: PrivateMessagingClient, request: ReadMessageRequest): Promise<ReadMessageResult>
listInbox(client: PrivateMessagingClient, request: ListMessagesRequest): Promise<ListMessagesResult>
listSent(client: PrivateMessagingClient, request: ListMessagesRequest): Promise<ListMessagesResult>
getMessageMetadata(client: PrivateMessagingClient, messageId: bigint | number | string): Promise<MessageMetadata>
getAccountStats(client: PrivateMessagingClient, account: string): Promise<AccountStats>
```

### `sendMessage`

Sends an encrypted message to a recipient address.

- `to`: recipient wallet address
- `plaintext`: plaintext message body
- `maxChunkBytes`: optional chunk size override
- `gasLimit`: optional manual gas limit override

Important defaults and limits:

- default safe chunk size: `24` bytes
- default encrypted message gas limit: `8_000_000`
- self-sends are rejected before broadcast
- the zero address is rejected before broadcast
- messages that exceed the contract chunk limit are rejected before broadcast

Example:

```typescript
import { sendMessage } from "@coti-io/coti-sdk-private-messaging";

const result = await sendMessage(client, {
  to: "0xRecipient",
  plaintext: "hello from coti"
});
```

### `readMessage`, `listInbox`, and `listSent`

These helpers read individual messages or paginated inbox and sent-message pages. When `decrypt` is `true` or omitted, the SDK attempts client-side decryption with the configured runner.

```typescript
import {
  listInbox,
  listSent,
  readMessage
} from "@coti-io/coti-sdk-private-messaging";

const inbox = await listInbox(client, {
  account: wallet.address,
  limit: 10
});

const sent = await listSent(client, {
  account: wallet.address,
  limit: 10,
  decrypt: false
});

const message = await readMessage(client, {
  messageId: 1n
});
```

## Rewards APIs

The rewards helpers expose the private messaging reward system:

```typescript
getContractConfig(client: PrivateMessagingClient): Promise<ContractConfig>
getCurrentEpoch(client: PrivateMessagingClient): Promise<bigint>
getEpochForTimestamp(client: PrivateMessagingClient, timestamp: bigint | number | string): Promise<bigint>
getEpochUsage(client: PrivateMessagingClient, epoch: bigint | number | string, agent: string): Promise<EpochUsage>
getEpochSummary(client: PrivateMessagingClient, epoch: bigint | number | string): Promise<EpochSummary>
getPendingRewards(client: PrivateMessagingClient, epoch: bigint | number | string, agent: string): Promise<bigint>
claimRewards(client: PrivateMessagingClient, request: ClaimRewardsRequest): Promise<ClaimRewardsResult>
fundEpoch(client: PrivateMessagingClient, request: FundEpochRequest): Promise<string>
```

Example:

```typescript
import {
  getCurrentEpoch,
  getEpochUsage,
  claimRewards
} from "@coti-io/coti-sdk-private-messaging";

const currentEpoch = await getCurrentEpoch(client);
const previousEpoch = currentEpoch - 1n;

const usage = await getEpochUsage(client, previousEpoch, wallet.address);

if (usage.pendingRewards > 0n && !usage.hasClaimed) {
  await claimRewards(client, { epoch: previousEpoch });
}
```

## Starter Grant APIs

The starter grant helpers interact with the external onboarding service:

```typescript
getStarterGrantChallenge(client: PrivateMessagingClient, config?: StarterGrantServiceConfig): Promise<GetStarterGrantChallengeResult>
getStarterGrantStatus(client: PrivateMessagingClient, config?: StarterGrantServiceConfig): Promise<GetStarterGrantStatusResult>
claimStarterGrant(client: PrivateMessagingClient, config: StarterGrantServiceConfig | undefined, input: ClaimStarterGrantRequest): Promise<ClaimStarterGrantResult>
requestStarterGrant(client: PrivateMessagingClient, config?: StarterGrantServiceConfig): Promise<RequestStarterGrantResult>
```

The SDK defaults to the deployed starter-grant service, so configuration is optional unless you want to override the service URL, timeout, auth token, or install ID path.

## MCP-style tools

The package also exports a JSON-safe tool registry and dispatcher:

```typescript
import {
  PRIVATE_MESSAGING_MCP_TOOLS,
  invokePrivateMessagingTool
} from "@coti-io/coti-sdk-private-messaging";

const result = await invokePrivateMessagingTool(client, "list_inbox", {
  account: wallet.address,
  limit: 10,
  decrypt: true
});
```

The MCP tool surface includes:

- `send_message`
- `read_message`
- `list_inbox`
- `list_sent`
- `get_contract_config`
- `get_account_stats`
- `get_message_metadata`
- `get_current_epoch`
- `get_epoch_for_timestamp`
- `get_epoch_usage`
- `get_pending_rewards`
- `get_epoch_summary`
- `claim_rewards`
- `fund_epoch`
- `get_starter_grant_challenge`
- `get_starter_grant_status`
- `claim_starter_grant`
- `request_starter_grant`

## MCP server

The package also ships with a stdio MCP server entrypoint:

```bash
npm run build
npm run start:mcp
```

Required environment variables:

- `PRIVATE_KEY`
- `AES_KEY`
- `COTI_NETWORK`

Optional overrides:

- `PRIVATE_MESSAGING_CONTRACT_ADDRESS_OVERRIDE`
- `COTI_RPC_URL_OVERRIDE`
- `COTI_TESTNET_RPC_URL_OVERRIDE`
- `COTI_MAINNET_RPC_URL_OVERRIDE`
- `STARTER_GRANT_SERVICE_URL`
- `STARTER_GRANT_SERVICE_TIMEOUT_MS`
- `STARTER_GRANT_SERVICE_AUTH_TOKEN`
- `STARTER_GRANT_INSTALL_ID_PATH`
