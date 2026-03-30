# COTI Starter Grant

This skill handles one-time gas funding for a newly created wallet.

## Overview

New wallets cannot interact with COTI until they have some native COTI for gas. The starter grant skill wraps the challenge-response flow used to fund an eligible wallet for the first time.

## Prerequisites

- the private messaging MCP server must be connected
- a wallet must already be configured
- the starter grant service must be available to the MCP server

## Typical workflow

### Recommended flow

1. Call `request_starter_grant`.
2. Check that the result status is `claimed`.
3. Confirm the wallet balance through a transaction or balance tool.

### Manual flow

1. Call `get_starter_grant_status`.
2. If eligible, call `get_starter_grant_challenge`.
3. Answer the lightweight challenge.
4. Call `claim_starter_grant`.

## Tool reference

### `request_starter_grant`

Runs the full challenge and claim sequence in one call.

### `get_starter_grant_status`

Returns one of:

- `eligible`
- `challenge_pending`
- `claimed`

### `get_starter_grant_challenge`

Returns the challenge prompt, challenge ID, claim payload, and expiry.

### `claim_starter_grant`

Signs the claim payload with the configured wallet and submits the claim.

## Common failures

- the wallet already claimed its one-time grant
- the challenge expired
- the starter grant service is unreachable
- the local install ID state is inconsistent

## Important notes

- the grant is one-time per wallet
- the challenge is intentionally simple
- the install ID is a soft deduplication signal, not a trustless identity primitive
- after claiming, the wallet should have enough COTI for initial messaging activity
