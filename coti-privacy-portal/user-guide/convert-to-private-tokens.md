# Convert to Private Tokens

The Privacy Portal lets you mint **privacy‑preserving** counterparts of approved public assets and later burn those private balances to reclaim the public custody held by each **privacy bridge** contract. Depending on the asset:

* **Native COTI → p.COTI:** You send native COTI in one transaction. The bridge applies a **protocol fee** in COTI and mints private tokens for the **net** amount (after the fee). Advanced users can also send COTI to the native bridge contract **without** the Portal UI; that path still uses the oracle for fees but does not pin the same **timestamp binding** the Portal uses when you go through an explicit deposit call.
* **ERC‑20 → private ERC‑20:** You move the **full** token amount into the bridge **via Portal In / the bridge `deposit` call**. The **protocol fee** is paid **separately in native COTI** in the same transaction—ensure you have enough **COTI** for that fee and for gas.

{% hint style="warning" %}
**Do not send ERC‑20 tokens with a plain wallet `transfer` to the bridge address.** Standard ERC‑20 transfers **do not** call the bridge’s code: **no** private tokens are minted, **no** fee is taken, and **no** `Portal In` accounting runs. Those tokens only increase the contract balance (like a stray transfer). Always use **Approve** + **Portal In** (or an integrator that calls **`deposit`** with the correct native fee and oracle timestamps).
{% endhint %}

Converting **ERC‑20** assets uses a two-step approval process in the UI:

{% stepper %}
{% step %}
#### Authorize the Conversion

If you are depositing ERC20 tokens (not native COTI), you must first approve the Portal to access your tokens:

* Select your desired asset from the Public Tokens list and enter the amount you wish to convert. Click **Approve**.
* Your MetaMask wallet will request a spending cap authorization. Review the estimated network fee and **confirm** the allowance.

<div align="left" data-with-frame="true"><figure><img src="../../.gitbook/assets/image (3)" alt="Approve Spending"><figcaption><p>Approve Spending</p></figcaption></figure></div>
{% endstep %}

{% step %}
#### Convert to Private

* After the spending approval is processed on-chain (when required), click the **Portal In** button.
* Confirm the minting transaction in your wallet. For **ERC‑20** deposits, attach **enough native COTI** for the quoted **bridge fee** as well as gas. For **native COTI** deposits, the UI sends one value: the bridge deducts the **protocol fee** and mints private tokens for the remainder.
* Your new minted Private Tokens will appear in the **Private Tokens** dashboard.

{% hint style="info" %}
Bridge fees use an on-chain **oracle** (Band via the price consumer). The Portal normally shows a quote and passes **oracle timestamps** so the price used at execution matches what you saw when estimating. If the oracle updates before your transaction lands, the transaction may **revert**—refresh the quote and try again.
{% endhint %}

<div align="left" data-with-frame="true"><figure><img src="../../.gitbook/assets/out.png" alt="Private Tokens"><figcaption><p>Private Tokens</p></figcaption></figure></div>
{% endstep %}
{% endstepper %}
