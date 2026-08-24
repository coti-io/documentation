# Transaction economics

## Built for real transaction economics

Confidentiality usually arrives with a size problem. FHE-based tokens attach a zero-knowledge input proof to every encrypted value — commonly **16–20 KB per input** — and offload multi-kilobyte ciphertext blobs off-chain.

PoD's encrypted values are small enough to treat like ordinary transaction data.

|                  | **PoD ERC-7984**     | **FHE-based confidential tokens**        |
| :--------------- | :------------------- | :--------------------------------------- |
| Encrypted input  | **~192 bytes**       | ~16,000–20,000 bytes                    |
| On-chain balance | **2 storage slots**  | Handle on-chain, multi-KB blob offloaded |
| Client-side work | **Encrypt and sign** | Generate a ZK proof, per transaction     |
| Numeric range    | **256-bit**          | 64-bit                                   |

**Roughly two orders of magnitude smaller on input.** Confidential transfers that fit comfortably inside normal block economics, on chains that were never designed for privacy.

## No proving in the browser

FHE inputs require the user's device to generate a zero-knowledge proof before a transaction can even be submitted — seconds of computation, and a frozen UI while it runs.

PoD asks the client to **encrypt and sign**. That is it. The wallet does what wallets already do, and the transaction goes out immediately. On mobile, on low-end hardware, and at scale, that difference compounds.
