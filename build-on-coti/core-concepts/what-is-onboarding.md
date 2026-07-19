# What Is Onboarding?

Onboarding is the one-time setup that gives your wallet its encryption key for private on-chain data. A short COTI Network transaction **retrieves** that key (your AES key) for your wallet address so you can encrypt and decrypt private on-chain values.

Until the key is available in your wallet session, you can still use the network normally (public transfers, gas, non-private contracts). You **cannot** read or write private data — for example private balances, amounts, or other encrypted contract variables.

When a dApp needs access to private data, it will ask you to complete onboarding by approving that transaction in your wallet.

{% hint style="info" %}
**Developers:** see [Account Onboarding Procedure](onboard-user.md) and [Account Onboard](../guides/account-onboard.md).
{% endhint %}

## Why do I need it?

Private values on COTI are stored encrypted. Encryption and decryption use a key that belongs only to **your** wallet account ([EOA](../../support-and-community/glossary.md#account)).

Your wallet needs that key to:

* **encrypt** inputs before private transactions
* **decrypt** outputs when you need to see private values

Without it, private features stay locked. The network already associates a unique AES key with your address; onboarding is how your wallet **receives** that key (first time) or **recovers** it again (new device, cleared storage, reinstall). Recovering always returns the **same** key for that address — it does not issue a new one.

## What is the encryption key?

* One AES key per wallet address — not shared across accounts
* Used only for **your** private data on COTI
* Held by your wallet / dApp session (for MetaMask users, often inside the [COTI Snap](../tools/coti-metamask-snap/README.md)); you usually never need to copy it
* Delivered wrapped so only your wallet can open it — not left as plaintext for others to read on-chain

{% hint style="danger" %}
Treat your AES key like your wallet private key. Anyone who has it can decrypt your private on-chain data. Never share your AES key, seed phrase, or private key.
{% endhint %}

Technical detail: [AES Keys](../../how-coti-works/advanced-topics/aes-keys.md).
