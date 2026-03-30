# Starter Grant

New wallets start with no native COTI, which means they cannot pay gas for their first transaction. The starter grant flow provides a one-time onboarding grant for eligible wallets.

## Overview

The starter grant is handled through an external service that works with the private messaging SDK and MCP server.

- each wallet can claim only once
- the flow is challenge-based
- the challenge is intentionally lightweight
- the SDK can complete the full request-and-claim cycle in one call

## Quick flow

The recommended path is `requestStarterGrant()`.

The SDK-level starter-grant helpers default to the deployed service, so `url` is optional unless you want to override it:

```typescript
import { requestStarterGrant } from "@coti-io/coti-sdk-private-messaging";

const claimA = await requestStarterGrant(client);

const claimB = await requestStarterGrant(client, {
  timeoutMs: 20000
});

const result = await requestStarterGrant(client, {
  url: "http://other-service:8787",
  timeoutMs: 15000
});
```

This helper:

1. requests a challenge
2. solves the lightweight prompt
3. signs the claim payload with the configured wallet
4. submits the final claim

The result includes:

- `status`
- `walletAddress`
- `transactionHash`
- `amountWei`
- `prompt`
- `expiresAt`

## Manual flow

If you want tighter control over the onboarding sequence, use the three-step flow:

```typescript
import {
  getStarterGrantStatus,
  getStarterGrantChallenge,
  claimStarterGrant
} from "@coti-io/coti-sdk-private-messaging";
```

1. Call `getStarterGrantStatus()` to check whether the wallet is `eligible`, `challenge_pending`, or `claimed`.
2. Call `getStarterGrantChallenge()` to receive `challengeId`, `prompt`, `claimPayload`, and `expiresAt`.
3. Call `claimStarterGrant()` with `challengeId`, `challengeAnswer`, and `claimPayload`.

## Service configuration

The starter grant helpers can use the deployed default service with no explicit configuration:

```typescript
await requestStarterGrant(client);
```

You can also pass optional overrides:

- `authToken`
- `installIdPath`
- `timeoutMs`
- `url`

Use `url` only when you want to override the default deployed service.

For MCP server deployments, the matching environment variables are:

- `STARTER_GRANT_SERVICE_URL`
- `STARTER_GRANT_SERVICE_TIMEOUT_MS`
- `STARTER_GRANT_SERVICE_AUTH_TOKEN`
- `STARTER_GRANT_INSTALL_ID_PATH`

## Common failures

Typical failures include:

- the starter grant service is not configured
- the wallet has already claimed the one-time grant
- the challenge expired before it was submitted
- the local install ID state is missing or inconsistent

## Important notes

- the grant is one-time per wallet address
- the wallet must exist before the claim flow starts
- the service uses an install ID as a soft local deduplication signal
- after a successful claim, the wallet usually has enough COTI for initial testnet activity such as messaging and contract interaction
