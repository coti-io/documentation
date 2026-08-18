# COTI private RWA implementation

A feature-level account of the ERC-3643 security token ported to COTI, deployed and executing on
testnet.

## 1. What is confidential now

Every row below is verified by a passing test against a live COTI node, not by inspection.


| Feature                          | Before                        | Now                       | How it works                                                                                                                                                                                          |
| -------------------------------- | ----------------------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Holder balances**              | Public`uint256`               | **Encrypted**             | Stored as`utUint256` — a contract copy plus a copy encrypted to the holder's key. The holder decrypts off-chain; nobody else can                                                                     |
| **Transfer amounts**             | Public in calldata and events | **Encrypted end to end**  | The amount enters as a signed`itUint256`, is computed on inside MPC, and leaves as **two ciphertexts** — one readable by the sender, one by the receiver                                             |
| **Allowances**                   | Public                        | **Encrypted, three ways** | `PrivateAllowance` holds a contract copy plus one each for owner and spender, so each party reads its own without a shared secret                                                                     |
| **Partially-frozen amounts**     | Public                        | **Encrypted**             | Eager copy for the holder; agents mint their own copy on demand via`reencryptFrozenTokens`                                                                                                            |
| **Mint and burn amounts**        | Public                        | **Encrypted in effect**   | The*requested* amount is public (an agent action), but the **effective** amount is computed under encryption — a burn larger than the holder's balance silently burns nothing, disclosing no balance |
| **Compliance evaluation**        | Public inputs                 | **Encrypted**             | `MaxBalancePrivateCompliance` keeps an encrypted per-holder shadow ledger and answers `canTransfer` without revealing the amount or the running total                                                 |
| **Whether a transfer succeeded** | Implicit in revert            | **Hidden**                | A blocked transfer moves an encrypted zero instead of reverting. See[§4](#4-moving-shares-to-another-participant) — this is the subtlest property here                                              |

---

## 2. What remains public, deliberately

Removing any of these would break the regulatory requirements.


| Public                                                    | Why                                                                                                                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Total supply**                                          | Fund size is disclosed in offering documents anyway, and a public total gives auditors a reconciliation anchor against encrypted per-holder balances    |
| **Identity and eligibility** (`isVerified`, country code) | A public regulatory fact. Encrypting it would defeat the point of ERC-3643                                                                              |
| **Compliance parameters** (caps, which modules are bound) | A regulator must be able to read the rulebook being enforced                                                                                            |
| **Agent roles, pause state, address-level freeze flag**   | Supervisory powers should be visible; only the*amount* frozen is secret, not the fact of a freeze                                                       |
| **The transaction graph**                                 | Addresses, timing and counterparties stay visible.**Amount confidentiality is not anonymity** — with ~30 holders, who traded with whom remains legible |

---

Navigation:
- [Private RWA Overview](./README.md)
- [COTI private RWA deployed contracts](./coti_rwa_deployments.md)
