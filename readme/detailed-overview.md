# Detailed Overview

Privacy solutions have been around for a while, but were always limited due to constraints in achieving full confidentiality and computational overhead. **COTI V2 introduces a novel approach that ensures performance without sacrificing privacy.**

To enable a performant solution with strong privacy guarantees, COTI V2 will utilize a novel combination of well-established privacy-preserving technologies (PETs), with the main ingredient being a Garbling Protocol.

### What are Garbling Protocols? <a href="#id-57b3" id="id-57b3"></a>

In the field of secure multi-party computation (MPC), a garbling protocol enables two or more parties to jointly compute a function while keeping both their inputs and intermediate variables private. Introduced initially in the 1980s, garbling protocols have become a cornerstone for privacy-preserving technologies.

### The Mechanism of Garbling Protocols <a href="#id-97b4" id="id-97b4"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/0*dyWaMbF7u_UV7av3" alt="" height="394" width="700"><figcaption></figcaption></figure>

The primary idea behind a garbling protocol is relatively straightforward, yet incredibly powerful. Imagine two parties, Alice and Bob, who wish to compute a function without revealing their inputs to each other. To do this, they use a Garbled Circuit, which works as follows:

* **Function Representation**: First, the function they wish to compute (which may be initially given as a mathematical formula or as a code written in high level programming language) is translated to a Boolean circuit. A Boolean circuit is a computational model that supports only a basic set of operations (also known as logical gates, like AND, OR, NOT) that can handle binary inputs (Booleans).
* **Circuit Garbling:** Alice, who takes the role of the ‘garbler,’ then encrypts or ‘garbles’ this circuit. However, instead of a traditional encryption, which works on _data_, garbling is an encryption that works on _functions_. The gates in the circuit and the inputs are encrypted in such a way that the output (of the circuit evaluation on the inputs) can only be understood if one has the correct ‘evaluation keys’.
* **Circuit Evaluation**: Following an interaction between Alice and Bob (known in the literature by oblivious transfer, or OT), in which Bob obtains these evaluation keys, Bob can evaluate the garbled circuit. As mentioned above, such evaluation will not leak to Bob the inputs or the intermediate variables; only the output of the function will be revealed.

### Advantages of Garbling Protocols <a href="#id-99cc" id="id-99cc"></a>

The primary advantage of garbling protocols is their ability to preserve the privacy of individual inputs while still allowing for joint computation. This makes them particularly useful in scenarios where confidential data needs to be computed upon but cannot be shared in plain, such as in privacy-preserving auctions, joint data analysis between organizations, or secure voting systems.

In the context of COTI V2, garbling protocols offer a revolutionary approach to handling transactional privacy. They can enable transactions and smart contract executions where the details (such as the amount of funds transferred, or the specific conditions of the contract) remain private between the involved parties. This level of privacy is particularly important in decentralized finance applications where transaction confidentiality can be as critical as transaction integrity.

### Novelty and comparison to other solutions <a href="#cc2a" id="cc2a"></a>

While other privacy solutions in Web3 are currently being developed, many of them rely on scaling technologies such as Zero-knowledge (ZK) cryptography to achieve transactional confidentiality.

Despite their effectiveness, ZK solutions aren’t without their limitations, especially when it comes to confidential transactions involving multiple parties. In this scenario, confidential data has to be stored and processed off-chain, either by the initiating user or a third party. This leads to an increase in centralization, as well as a dependency on what could potentially be an insecure storage solution. This is the case for privacy solutions like Secret, Obscuro, and Oasis, technologies that rely **fully** on secure enclaves such as Intel and SGX. Worse still, these enclaves haven’t been as secure as advertised, attracting numerous data breaches over the past few years.

Additionally, ZK proofs incur a significant cost for on-chain verification and offer a suboptimal user experience. At present levels, SNARK-proof verification costs around 200k gas on the EVM and may take up to a few seconds to compute on the client device.

COTI V2’s use of garbling protocols takes a completely novel approach. Unlike ZK solutions which support a single data source (or owner), and are expensive on the client side, garbling protocols enable computation on private data coming from many sources (or owners). Furthermore, they’re able to maintain private storage in addition to Ethereum’s standard public global state.

Garbling protocols already allow for a very efficient client, however, with breakthroughs made by COTI V2, the technology is up to **ten times lighter and performs ten times faster** than ZK based solutions without negatively impacting the user experience. This means that the **technology has the ability to run on almost any device, expanding the range of potential use cases in the future.** Adopting a privacy solution that supports operation on private data from multiple sources also helps in mitigating Maximal Extractable Value (MEV) losses, as a portion of crucial data remains encrypted at all times.

When it comes to the security of the network, COTI V2 is predicated on the fact that a threshold of the network nodes operate faithfully. While COTI plans to use secure enclaves for an additional layer of protection, they are not a necessity for the integrity of the system. Additionally, this design allows users to withdraw directly from L1 without needing L2’s permission, further emphasizing the secure nature of COTI V2.

COTI V2’s cryptographic protocols are both secure and efficient, allowing parties to jointly evaluate functions over their private data without leaking information. What sets COTI V2 apart is its unique integration of a garbling protocol within a larger MPC solution to facilitate confidential transactions and smart contract interactions on a layer 2 platform. This synergy not only enhances privacy by ensuring inputs remain encrypted throughout the computation process, but also significantly reduces the risk of data exposure or manipulation by malicious actors.

\
