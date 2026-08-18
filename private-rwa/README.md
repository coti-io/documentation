# Private RWA

This section covers COTI's private RWA implementation and its live deployment details.

1. COTI Private ERC-3643 design

**The rulebook** is ERC-3643's on-chain compliance configuration: the `IdentityRegistry` deciding who may hold the token, the `ModularCompliance` contract and its bound modules deciding what constrains a transfer, and the agent roles, pause, and freeze controls above them. It is the machine-readable form of the transfer restrictions an offering document states in prose-allowing regulators and auditors to verify compliance by reading the code rather than trusting the issuer.

The COTI Private ERC-3643 design draws a clear line between a *rule* and the *position measured against it*: **the rulebook stays public; the amounts do not.** A regulator, auditor, or counterparty can still view who is eligible to hold the token, which compliance rules are bound and with what parameters, and how large the overall fund is. What they cannot view is any individual position, transfer size, allowance, or frozen amount.

That split is deliberate. Encrypting the rulebook would destroy the auditability that defines ERC-3643; leaving amounts public exposes investor balances and wallet-level ownership distribution.

Navigation:

- [COTI private RWA implementation](./coti_rwa.md)
- [COTI private RWA deployed contracts](./coti_rwa_deployments.md)
