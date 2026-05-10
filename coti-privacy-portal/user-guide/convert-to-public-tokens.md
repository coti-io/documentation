# Convert to Public Tokens

The Private Tokens dashboard offers **Portal Out**, which converts private tokens back into standard public tokens. The portal uses the same pattern as Portal In: when the bridge needs permission to move your **private** balance, you **Approve** first, then confirm the withdrawal.

{% stepper %}
{% step %}
#### Unlock and choose the asset

* Ensure your private balances are unlocked (see [Setup Portal Account — Unlock private tokens](setup-portal-account.md)).
* Select the asset from your **Private Tokens** list and enter the amount you want to withdraw.

<div align="left" data-with-frame="true"><figure><img src="../../.gitbook/assets/Screenshot 2026-03-24 at 3.42.27 PM.png" alt="Portal Out"><figcaption><p>Portal Out</p></figcaption></figure></div>
{% endstep %}

{% step %}
#### Approve (when the portal asks)

* If the modal shows an **Approve** step, confirm it in MetaMask. This grants the privacy bridge permission to spend the **private** token amount you are withdrawing (encrypted allowance), similar to approving a public ERC-20 before Portal In.
* Wait until the approval transaction is confirmed. The UI will then let you continue to the actual withdrawal.

{% hint style="info" %}
Native **p.COTI** withdrawals also use an approval step for the native privacy bridge when required by the app flow.
{% endhint %}
{% endstep %}

{% step %}
#### Confirm Portal Out (withdraw)

* Click **Portal Out** (or the button shown to complete the withdrawal) and confirm the transaction in MetaMask.
* Private tokens are **burned** for the amount you withdraw. Your wallet then receives **public** tokens matching that flow (**native COTI** or the **ERC‑20** for that asset), subject to bridge rules below.

{% hint style="info" %}
**Native p.COTI → public COTI:** On-chain you burn the **full** private amount you approved. The bridge takes the **protocol fee** in native COTI from that withdrawal, so **public COTI you receive ≈ burnt amount − fee** (the fee stays with the bridge until collected by the protocol). The Portal should quote the fee from the oracle before you sign.

**Private ERC‑20 → public ERC‑20:** You receive the **full** public token amount shown for the withdrawal. The bridge **protocol fee** is charged **separately in native COTI** (`msg.value` in the transaction), not withheld from the token amount—have enough native COTI balance for gas and that fee.

**Smart contract wallets:** If your wallet rejects **unsolicited ETH** from the bridge, any **refunded** native COTI may be credited for you to **claim** yourself later via the bridge helper (same **`msg.sender` only** — use an address or flow that accepts native transfers).
{% endhint %}

    <div data-with-frame="true"><figure><img src="../../.gitbook/assets/Screenshot 2026-03-24 at 3.42.27 PM.png" alt="Transaction Approval"><figcaption><p>Confirm withdrawal in the wallet</p></figcaption></figure></div>

    <br>
{% endstep %}
{% endstepper %}
