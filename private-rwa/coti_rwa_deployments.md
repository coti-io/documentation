# COTI private RWA Spec

## 7.COTI Private RWA Deployed contracts

[rwa.demo.coti.io](https://rwa.demo.coti.io) Contracts are live on COTI testnet (chain `7082400`). The fund contracts and the test-suite contracts were deployed from [`0xAb81c57CCc578a5636BFF47B896BEC6Af1c30012`](https://testnet.cotiscan.io/address/0xAb81c57CCc578a5636BFF47B896BEC6Af1c30012); Every address links to its page on [testnet.cotiscan.io](https://testnet.cotiscan.io).

### The token is not the contract you transact with

Each fund has **two** contracts :

- **`PrivateToken`** is the *asset* — it holds your encrypted share balance. You never send it a
  transaction directly.
- **`RwaSubscription`** is the *till* — you call `subscribe(paymentToken, amount)` here, it pulls
  your USDC or USDT, computes `amount × 1e8 / price`, and mints the shares on the token.

So the address in your wallet history is the **subscription** contract, while your balance lives on
the **token**. Because the mint is an internal call, the token's explorer page shows no transaction
for your purchase at all — it appears under *Internal txns*. `RwaSubscription` has no `symbol()`;
importing it into a wallet as a token will fail. The link runs one way: `subscription.token()`
returns the token address, and the token knows nothing about the subscription beyond having granted
it agent rights to mint.

### The demo fund stack

Each fund gets its own four contracts — nothing is shared between funds except the onboarding contract and the payment tokens.

**These are  the addresses [rwa.demo.coti.io](https://rwa.demo.coti.io) is configured against**, checked against the chain: every `token`, `registry`, `compliance` and `subscription


| Fund  | Role              | Contract                      | Address                                                                                                                        |
| ----- | ----------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| JTRSY | Share token       | `PrivateToken`                | [`0x6D7cf587dbF68eb233B7BEd1f45BDfB6aE31Baf3`](https://testnet.cotiscan.io/address/0x6D7cf587dbF68eb233B7BEd1f45BDfB6aE31Baf3) |
| JTRSY | Subscription      | `RwaSubscription`             | [`0x5cf23F0cf6369477d1F267e5f9F281C3e6B8A98f`](https://testnet.cotiscan.io/address/0x5cf23F0cf6369477d1F267e5f9F281C3e6B8A98f) |
| JTRSY | Identity registry | `MockPrivateIdentityRegistry` | [`0x9Da490afb22cEb1B8aA82d2EC4418BB4A62e5F37`](https://testnet.cotiscan.io/address/0x9Da490afb22cEb1B8aA82d2EC4418BB4A62e5F37) |
| JTRSY | Compliance        | `MaxBalancePrivateCompliance` | [`0xB5d2e8880005dCF84f13Fc58626d7F67734E28CB`](https://testnet.cotiscan.io/address/0xB5d2e8880005dCF84f13Fc58626d7F67734E28CB) |
| JAAA  | Share token       | `PrivateToken`                | [`0x20b2C3cc4F7b4a5f727b1aa69779aD9C20036732`](https://testnet.cotiscan.io/address/0x20b2C3cc4F7b4a5f727b1aa69779aD9C20036732) |
| JAAA  | Subscription      | `RwaSubscription`             | [`0x0A1089dc8b71E463c3AD89363058B5a07A7f1bf9`](https://testnet.cotiscan.io/address/0x0A1089dc8b71E463c3AD89363058B5a07A7f1bf9) |
| JAAA  | Identity registry | `MockPrivateIdentityRegistry` | [`0xC64DC85109E823380ea4DE34b6ac1B22a02Ba23E`](https://testnet.cotiscan.io/address/0xC64DC85109E823380ea4DE34b6ac1B22a02Ba23E) |
| JAAA  | Compliance        | `MaxBalancePrivateCompliance` | [`0x2abfd1194120fb2BDc2D3Fd8366C2979c7aea531`](https://testnet.cotiscan.io/address/0x2abfd1194120fb2BDc2D3Fd8366C2979c7aea531) |
| —    | Onboarding        | `AccountOnboard`              | [`0x536A67f0cc46513E7d27a370ed1aF9FDcC7A5095`](https://testnet.cotiscan.io/address/0x536A67f0cc46513E7d27a370ed1aF9FDcC7A5095) |

Both funds issue shares at 8 decimals. Subscription prices are held per payment token, in payment-token units per `1e8` shares: JTRSY `1_112_439` (1.112439), JAAA `1_044_450` (1.04445). `quote()` floors, so a subscription of 1100 USDC.e into JAAA mints 1053.18588730 shares.

### Payment tokens

Plain ERC-20s with public balances and amounts are exchanged by RwaSubscription to mint RWA tokens. The stablecoin leg of a subscription is fully visible**; only the share balance
that comes back is encrypted. These are the only two payment tokens supported:


| Token        | Symbol   | Decimals | Address                                                                                                                        |
| ------------ | -------- | -------: | ------------------------------------------------------------------------------------------------------------------------------ |
| Bridged USDC | `USDC.e` |        6 | [`0x63f3D2Cc8F5608F57ce6E5Aa3590A2Beb428D19C`](https://testnet.cotiscan.io/address/0x63f3D2Cc8F5608F57ce6E5Aa3590A2Beb428D19C) |
| Tether USD   | `USDT`   |        6 | [`0x9e961430053cd5AbB3b060544cEcCec848693Cf0`](https://testnet.cotiscan.io/address/0x9e961430053cd5AbB3b060544cEcCec848693Cf0) |

### The application that drives it

**The following table displays [rwa.demo.coti.io](https://rwa.demo.coti.io) application flow, and where confidentiality starts and stops:**


| Step           | What happens                                                                                                                                          | Confidential?                                                                             |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 1. Connect     | RainbowKit; the app offers to add COTI Testnet (`7082400`) if the wallet is elsewhere                                                                 | n/a                                                                                       |
| 2. Unlock      | The plugin prompts for a **wallet signature** and derives the account's AES key from it, held for the session. **The app never handles a private key** | The key never leaves the browser                                                          |
| 3. Eligibility | `isVerified(address)`, and `setVerified` on the mock registry if needed                                                                               | **No** — a public regulatory fact by design ([§3](#3-what-remains-public-deliberately)) |
| 4. Subscribe   | `approve` on USDC.e or USDT, then `RwaSubscription.subscribe(paymentToken, amount)`                                                                   | **No** — see the disclosure below                                                        |
| 5. Hold        | `balanceOf` returns a `ctUint256`, decrypted **in the browser** with the session key                                                                  | **Yes** — this is the thesis, visible                                                    |

Navigation:

- [Private RWA Overview](./README.md)
- [COTI private RWA implementation](./coti_rwa.md)
