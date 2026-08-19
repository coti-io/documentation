# PoD pERC20 vs ERC7984

**Confidential tokens on the chains you already use.**

Privacy on Demand brings encrypted balances and encrypted transfers to any EVM chain — no privacy-native L1, no specialised rollup, no migration. Live today on Avalanche Fuji and Ethereum Sepolia.

---

## Wrap the assets that actually exist

PoD pERC20 is a 1:1 collateralised confidential wrapper. Lock WETH, get `p.ETH`. Lock USDC, get `p.USDC`. The private token mirrors the underlying exactly — same decimals, same supply, same value — and unwraps back on demand.

That "same decimals" part is where PoD stands alone.

**FHE-based confidential token standards store balances as 64-bit encrypted integers.** At 18 decimals, a 64-bit ceiling caps a token at roughly **18.4 whole units** before it overflows. A confidential 1:1 WETH wrapper is not difficult under that constraint — it is arithmetically impossible above ~18 ETH.

PoD carries **full 256-bit precision** end to end: 18 decimals, `uint256` range, no ceiling worth naming. Four of the six pTokens live today are 18-decimal, including `p.ETH` and `p.AVAX`.

> **If you want to wrap real liquidity confidentially, 256-bit is not a preference. It is the entry requirement.**

---

## Deploy anywhere there is an Inbox

Confidential tokens have historically meant moving to a privacy chain and asking your users to follow. PoD inverts that: the token lives on **your** chain, and the encrypted computation happens on COTI behind the scenes.


|                                |                                                                 |
| :----------------------------- | :-------------------------------------------------------------- |
| **Chains supported**           | Any EVM chain with a PoD Inbox deployed                         |
| **Live today**                 | Avalanche Fuji, Ethereum Sepolia                                |
| **Required of the host chain** | Nothing — no FHE precompiles, no custom opcodes, no forked EVM |
| **Required of the user**       | A standard wallet                                               |

Your liquidity, your users, and your existing integrations stay exactly where they are.

---

## Built for real transaction economics

Confidentiality usually arrives with a size problem. FHE-based tokens attach a zero-knowledge input proof to every encrypted value — commonly **16–20 KB per input** — and offload multi-kilobyte ciphertext blobs off-chain.

PoD's encrypted values are small enough to treat like ordinary transaction data.


|                  | **PoD pERC20**       | **FHE-based confidential tokens**        |
| :--------------- | :------------------- | :--------------------------------------- |
| Encrypted input  | **~192 bytes**       | ~16,000–20,000 bytes                    |
| On-chain balance | **2 storage slots**  | Handle on-chain, multi-KB blob offloaded |
| Client-side work | **Encrypt and sign** | Generate a ZK proof, per transaction     |
| Numeric range    | **256-bit**          | 64-bit                                   |

**Roughly two orders of magnitude smaller on input.** Confidential transfers that fit comfortably inside normal block economics, on chains that were never designed for privacy.

---

## No proving in the browser

FHE inputs require the user's device to generate a zero-knowledge proof before a transaction can even be submitted — seconds of computation, and a frozen UI while it runs.

PoD asks the client to **encrypt and sign**. That is it. The wallet does what wallets already do, and the transaction goes out immediately. On mobile, on low-end hardware, and at scale, that difference compounds.

---

## Precise, confidential approvals

PoD keeps the allowance model developers already know — and encrypts it.

- **Exact amounts.** Approve 50 tokens, not blanket authority over the balance.
- **Encrypted on-chain.** The allowance value is a ciphertext, readable only by the owner and the spender.
- **Standard semantics.** `approve` / `transferFrom`, the shape every integrator already knows.

Blanket time-boxed operator models grant a spender full authority over a balance until expiry, and record that authority publicly. PoD grants a specific encrypted amount, and keeps the amount private.

---

## Failure that reveals nothing

When an encrypted transfer exceeds a balance, PoD resolves it inside the garbled circuit: the effective amount becomes zero and the request completes normally. **No revert, no error code, no observable difference** between a transfer that moved value and one that did not.

Insufficient balances stay as private as sufficient ones.

---

## A security model without a master key

PoD's confidentiality rests on **garbled circuits with non-colluding parties** — a garbler and an evaluator that must both defect to compromise a session.

The distinction that matters is what a worst case costs you. Threshold FHE systems protect a shared decryption key; if enough key-holders collude, **every transaction ever recorded becomes readable retroactively**. PoD has no such key. A compromise is scoped to a session, not to your history.

**Privacy that does not accumulate a single point of catastrophic failure.**

---

## Live on two public testnets

Six confidential tokens, deployed and operating.

**Avalanche Fuji**


| Token    | Address                                      | Decimals |
| :------- | :------------------------------------------- | :------: |
| `p.MTT`  | `0x02f284a1968160E1d3e4bC2BA3261be49725E765` |    18    |
| `p.USDC` | `0x21576D8CCE47d044C5815bd59eca1F6DA94c65A5` |    6    |
| `p.AVAX` | `0x74d47cD68203066c97BA99787Fe1e0c68Ce42b04` |    18    |

**Ethereum Sepolia**


| Token    | Address                                      | Decimals |
| :------- | :------------------------------------------- | :------: |
| `p.MTT`  | `0x0510F0b32828D5fB472dE5A5bE30b370c5D1a056` |    18    |
| `p.USDC` | `0xD7B3D49F85000489708B7db5B0f1a8693Fc707f3` |    6    |
| `p.ETH`  | `0xd33A363459c6Ee0C4F8504E380E8D3Aa4F209116` |    18    |

Each pair is deployed by `PrivacyPortalFactory` as a minimal-proxy clone — **one portal and one pToken per asset**, so listing a new confidential token is a factory call, not an engineering project.

---

## Explorer-native from day one

PoD pTokens emit `ConfidentialTransfer` events and expose ERC-7984 metadata, so Blockscout and compatible explorers classify them as confidential tokens and index their activity automatically. Your users see a first-class token page, not an unrecognised contract.

---

## Concurrency that keeps up with users

Multiple transfers, mints, and burns from the same account can be in flight simultaneously. Each is tracked independently by request id, and a monotonic nonce guarantees results always apply in the correct order. **No queue, no serialisation, no waiting for one transfer to clear before starting the next.**

---

## Why teams choose PoD pERC20


|                              |                                                                           |
| :--------------------------- | :------------------------------------------------------------------------ |
| **Wrap real assets**         | 18-decimal confidential WETH and WAVAX — out of reach for 64-bit designs |
| **Keep your chain**          | Runs on any EVM chain; no privacy L1, no migration                        |
| **Full precision**           | 256-bit values, no supply ceiling                                         |
| **Small on-chain footprint** | ~192-byte inputs, 2-slot balances                                         |
| **Instant for users**        | No client-side proof generation                                           |
| **Private allowances**       | Exact encrypted amounts, not public blanket authority                     |
| **No master key**            | Session-scoped security, no retroactive exposure                          |
| **Ship fast**                | Factory-deployed portal + token pair per asset                            |

---
