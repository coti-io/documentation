# Privacy on Avalanche (PoD) for Dummies

![Pod for Dummies Infographics](../.gitbook/assets/pod-infographics.png)

## Why care about privacy on a public blockchain?

Public blockchains such as **Avalanche** are designed for transparency. Every transaction, contract call and balance is visible to anyone. This openness provides agreement, auditability and open participation, but it comes at a cost: sensitive information can leak. There are many situations where you **do not** want to expose the details of a transaction or an internal computation – for example, when handling salaries, bidding in an auction, running a confidential vote or computing a credit score. Traditional options are limited: you either keep the sensitive data off‑chain (breaking composability) or move everything to a specialized privacy chain (which may not fit your liquidity or regulatory needs).

**Privacy on Avalanche** applies **Privacy on Demand (PoD)** to **Avalanche Fuji C-Chain**. Keep wallets, AVAX, and dApp contracts on Avalanche while off‑loading **sensitive computation** to COTI. Applications send **encrypted inputs** to COTI, receive **encrypted outputs** back on Fuji, and decrypt them locally. In other words: **Avalanche for coordination**, **COTI for the sensitive math and data handling**.

## What is PoD?

PoD is a pattern for **private computation on COTI** while continuing to use your existing blockchain for tokens and business logic. In simple terms:

* **Privacy on Avalanche / PoD** – A way to do calculations secretly on COTI while keeping accounts and assets on **Avalanche Fuji**.
* **Inbox** – An on‑chain mailbox. It forwards your encrypted request to COTI and brings back the answer. Think of it as a certified courier carrying sealed letters.
* **MPC executor** – COTI’s private calculator. It runs the secret maths and returns encrypted results.

The sequence is straightforward:

1. **Stay on Avalanche.** Your application continues to run on **Avalanche Fuji C-Chain** just as it does today.
2. **Encrypt sensitive data.** The user’s app turns the input into an encrypted parcel before sending it. The cleartext never appears on the chain.
3. **Send via the Inbox.** The application forwards the parcel through the Inbox to COTI.
4. **Compute privately on COTI.** COTI performs the secret computation and produces an encrypted answer.
5. **Get the result and decrypt it.** The encrypted answer comes back through the Inbox. Only the user can open it with their key.

This process hides amounts, scores or business rules while still benefiting from the public blockchain’s security and composability.

## How does a PoD request travel end to end?

To understand PoD, let’s follow a request from the user’s perspective without getting into programming details. There are a few key actors in the story: the **End user**, the **client app**, **your dApp contract**, and the **PoD infrastructure** that connects your chain with COTI. Each plays a different part in the journey:

| Actor                  | Role                                                                                                                                                                                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **End user**           | A person using a wallet or app who approves transactions and holds the decryption key.                                                                                                                                                                                                     |
| **Client app**         | A browser or mobile app that encrypts inputs and later decrypts the result on the user’s device.                                                                                                                                                                                           |
| **Your dApp contract** | A smart contract on your chain that defines business rules, stores correlation IDs and holds encrypted results.                                                                                                                                                                            |
| **PoD infrastructure** | The combination of the Inbox and the private compute engine on COTI. It forwards your encrypted request to COTI, performs the secret calculation and returns an encrypted result back to your contract. In other words, it is the bridge and calculator that connects two chains together. |

The **journey** happens in seven steps:

1. **User chooses an action.** This could be comparing two numbers, making a private payment or running a private score.
2. **Client encrypts inputs.** The app turns the user’s data into an encrypted `it*` payload; the plaintext never hits the chain.
3. **User submits an EVM transaction.** The dApp contract checks permissions, payments and other rules.
4. **Contract talks to the PoD infrastructure.** Your contract sends the encrypted job through the PoD infrastructure – the message is forwarded to COTI and a promise to return the answer is recorded.
5. **PoD infrastructure computes and returns.** On the COTI side, the private engine performs the secret computation and then the PoD infrastructure delivers the encrypted result back to your contract. You don’t see the intermediate steps; it’s all handled behind the scenes.
6. **User decrypts the result.** The client fetches the ciphertext and decrypts it locally with their account key.

Because private execution happens on another chain, PoD is **asynchronous**: the result does not arrive in the same transaction. Users should think of it like submitting a job and waiting for a callback. UI designers should show _Pending_, _Completed_ and _Failed_ states, and teams need indexing to match callbacks to requests. Decryption always happens client‑side; no plaintext is ever written on‑chain.

![Infographics](../.gitbook/assets/pod-infographics-2.png)

## Behind the scenes

You don’t need to be a developer to understand the basic parts involved. Think of the system as three rooms connected by a mail slot:

1. **Your device.** This is your wallet or app. It holds your keys and encrypts the information you want to keep private.
2. **Avalanche Fuji C-Chain.** This is the host network where your tokens and smart contracts live (fees in **AVAX**). It includes the **Inbox**, which is just a mailbox that forwards your encrypted request and later delivers the reply.
3. **The private compute room (COTI).** This is COTI’s secure environment. Inside, a private engine performs the computation and produces an encrypted answer.

When you use PoD, you send a sealed envelope through the mailbox. The envelope travels to the private room, where a trusted party reads the encrypted contents, performs the requested calculation and seals the result. The sealed reply comes back through the mailbox. Only you can open it because you hold the key. All of the routing and encryption details are handled by the underlying tools – there’s no need to understand smart contracts or special libraries.

### Optional: Your dApp’s private execution on COTI

For many applications, the built‑in PoD operations are enough – you can compare values, add numbers or perform common checks without writing any special code. However, if your dApp needs unique functionality (for example, a bespoke scoring algorithm, a secret auction mechanism or a complex data transformation), there is an option to run your **own private logic** on COTI. In this section – **Your dApp’s private execution on COTI** – we outline how developers can extend PoD beyond the standard operations:

* **Custom private code:** Developers can deploy a program on COTI that defines the logic for their private computation. It only accepts encrypted inputs and returns encrypted outputs, so its inner workings and data remain hidden from everyone else.
* **Connected by PoD infrastructure:** Your dApp contract still sends encrypted requests through the PoD infrastructure. The infrastructure routes them to your private program on COTI and brings back the result.

From a user’s perspective, nothing changes – you still submit a job and receive an encrypted answer. Behind the scenes, though, your dApp’s “secret sauce” runs privately on COTI. This option allows businesses to implement privacy‑preserving features tailored to their needs without exposing sensitive logic or data. In short, **PoD infrastructure** remains your bridge between chains, but you can choose to host your app’s private components on COTI rather than relying solely on the built‑in operations.

## What can you use PoD for?

PoD unlocks a range of applications that were previously difficult or impossible on public blockchains. Here are a few examples:

* **Confidential payments:** Private transfers of tokens or stablecoins where only the sender and receiver know the amount. Useful for payroll, royalties and peer‑to‑peer payments.
* **Fair auctions and secret bids:** Run sealed‑bid auctions where bids remain hidden until the auction closes. This prevents front‑running and bid collusion.
* **Private DeFi strategies:** Execute trades or calculate liquidations without revealing positions or strategies to competitors. PoD can shield orders from MEV bots and protect traders from being sandwiched.
* **Secure credit scoring and KYC checks:** Perform scoring or identity verification on encrypted data. A dApp can check that a user’s credit score exceeds a threshold without seeing the raw score.
* **Confidential voting and governance:** Cast votes privately and prove that votes were counted correctly without exposing individual choices. Ideal for DAOs, corporate boards or community governance.
* **Secret games and NFTs:** Reveal the outcome of a game or the attributes of an NFT only to the owner, keeping the randomness or rarity hidden until needed.

These examples share a common pattern: sensitive inputs stay encrypted, computation happens in a private environment and only authorized parties see the outputs. PoD makes these patterns accessible to developers building on EVM chains.

## Why PoD operations are asynchronous

In a typical smart‑contract interaction you send a transaction and immediately see the result. PoD is different because private execution happens off‑chain in COTI’s secure environment. The call is **asynchronous**: you submit a job, then wait for a callback containing the encrypted result. There are three phases:

1. **Request transaction:** your dApp contract submits the encrypted job through the Inbox and receives a request ID.
2. **Wait:** COTI performs the private computation. This may take a few seconds or longer, depending on network conditions.
3. **Callback transaction:** the Inbox calls your contract’s callback function with the encrypted output.

Because of this asynchronous nature, the user experience must include pending/completed/failed states and error handling. After completion, the plaintext result is **not** public on‑chain; the user’s client decrypts it locally. Thinking of PoD as “send a job, get a sealed reply” helps set expectations.

## Why is there a fee?

Every PoD call uses resources on **two** networks: **Avalanche Fuji** and the COTI chain. When you send a two‑way message through the Inbox, you pay a fee in **AVAX**. This fee funds two things:

1. **Private computation on COTI.** The MPC executor executes the garbled‑circuit logic and relays the request. This consumes gas and relayer work on the COTI side.
2. **Callback on Fuji.** After computation, the Inbox must relay the result back and execute your contract’s callback function, consuming gas on Avalanche C-Chain.

In the worked Fuji example, a user sends **0.02 AVAX** as `msg.value` and reserves **0.008 AVAX** as `callbackFeeLocalWei` for the Fuji return path; the remaining **0.012 AVAX** funds the COTI leg after oracle conversion (AVAX/COTI prices). Because AVAX is typically much more valuable per token than COTI, even a small AVAX remote slice can become a large COTI gas budget — that is expected. Underfunding either side can leave the request stuck in a pending state: too little `msg.value` (or remote slice) hurts the outbound leg; too little `callbackFeeLocalWei` prevents the Fuji callback from writing the result. Details and arithmetic: [How do PoA fees work on Avalanche Fuji?](how-poa-fees-work.md).

## Benefits and limitations

**Benefits:**

* **Seamless integration:** PoD lets you add privacy to existing dApps without migrating assets or users. You keep your tokens and workflows on your favourite chain.
* **Strong confidentiality:** Sensitive inputs and outputs stay encrypted; computation happens in a secure enclave and only authorized clients decrypt results.
* **Flexible and composable:** Future tools let developers build simple or complex private operations without needing to rebuild everything from scratch. You won’t notice a difference; your app simply turns privacy on when needed.
* **Regulatory alignment:** PoD enables compliant confidentiality – data can be audited on the public chain while personal or business details remain protected.
* **Cross‑chain future:** COTI is building cross‑chain capabilities so that privacy can be turned on or off across many chains, further expanding its reach (as announced on Medium in Nov 2024).

**Limitations to consider:**

* **Not full anonymity:** PoD does not hide metadata such as gas costs, timing or which contract was called. A determined observer might infer patterns unless you design around them.
* **Asynchronous UX:** Users must wait for callbacks. UIs need to handle pending states and errors gracefully.
* **Fee planning:** Each leg of the job requires funding. Underestimating fees can leave jobs hanging.
* **Key management:** Users need to manage client‑side encryption keys. Losing a device or key can mean losing access to decrypted outputs.
* **Not legal advice:** PoD is a technology, not a substitute for regulatory compliance. Teams must consult legal experts for their specific use case.

## Conclusion

Privacy on Demand brings a pragmatic approach to confidential computing on public blockchains. By combining the openness of EVM chains with the privacy of COTI’s garbled‑circuit engine, builders can create applications that protect users’ confidentiality without sacrificing composability or liquidity. The pattern is simple: **encrypt, send via Inbox, compute privately, receive and decrypt**. The underlying developer tools handle the routing and cryptography so you don’t need to understand the underlying smart‑contract details. While PoD introduces new considerations (fees, asynchronous UX, key management), it unlocks a world of private payments, fair auctions, confidential DeFi and secure voting – all from the comfort of your existing blockchain stack.
