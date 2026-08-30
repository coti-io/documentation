# Transfer semantics

## Precise, confidential approvals

COTI keeps the allowance model developers already know from ERC-20 — and encrypts it. This is a deliberate alternative to ERC-7984’s time-boxed unlimited [operator model](compatibility-and-divergence.md).

- **Exact amounts.** Approve 50 tokens, not blanket authority over the balance.
- **Encrypted on-chain.** The allowance value is a ciphertext, readable only by the owner and the spender.
- **Standard semantics.** `approve` / `transferFrom`, the shape every integrator already knows (including ERC-2612-style permit flows where exposed).

Blanket time-boxed operator models grant a spender full authority over a balance until expiry, and record that authority publicly. COTI grants a specific encrypted amount, and keeps the amount private.

## Failure that reveals nothing

When an encrypted transfer exceeds a balance, COTI resolves it inside the garbled circuit: the effective amount becomes zero and the request completes normally. **No revert, no error code, no observable difference** between a transfer that moved value and one that did not.

Insufficient balances stay as private as sufficient ones.
