# Transfer semantics

## Precise, confidential approvals

PoD keeps the allowance model developers already know — and encrypts it.

- **Exact amounts.** Approve 50 tokens, not blanket authority over the balance.
- **Encrypted on-chain.** The allowance value is a ciphertext, readable only by the owner and the spender.
- **Standard semantics.** `approve` / `transferFrom`, the shape every integrator already knows.

Blanket time-boxed operator models grant a spender full authority over a balance until expiry, and record that authority publicly. PoD grants a specific encrypted amount, and keeps the amount private.

## Failure that reveals nothing

When an encrypted transfer exceeds a balance, PoD resolves it inside the garbled circuit: the effective amount becomes zero and the request completes normally. **No revert, no error code, no observable difference** between a transfer that moved value and one that did not.

Insufficient balances stay as private as sufficient ones.
