---
description: >-
  Secure operations are supported via our gcEVM extension, which is implemented
  through a set of precompiled contracts.
---

# Secure Operations and Gas

To bring a new private datum into the gvEVM the function's signature should include an argument of type `Inputtext`. Then, inside the function, it is necessary to call `ValidateCiphertext`, which returns a `Garbledtext` if verification succeeds and an error otherwise. Then, the contract is free to save private data temporarily in Memory, by calling `Offboard` on the relevant `Garbledtext.`This returns a private datum of type `Ciphertext` that awaits other commands, namely, the `Ciphertext` type cannot serve as an operand to secure operations, rather it should be `Onboarded` again and turn into a `Garbledtext` first; alternatively, the `Ciphertext` may be stored permanently by simply assigning it to a state variable of the contract.

## List of Operations and Their Required Gas

{% hint style="warning" %}
Each operation within the network requires a specific amount of gas. It's important to note that the following gas allocations are intended to be proportional and serve as initial estimates. The final determination of gas requirements will be made at a later stage, taking into account various factors such as network performance, usage patterns, and protocol optimizations.
{% endhint %}

List of operations supported on `Garbledtext`and their required gas prices

<table data-full-width="false"><thead><tr><th width="133">Operation</th><th>Bool</th><th>gtUint8</th><th>gtUint16</th><th>gtUint32</th><th>gtUint64</th></tr></thead><tbody><tr><td><code>And</code></td><td>12005</td><td>12040</td><td>12080</td><td>12160</td><td>12320</td></tr><tr><td><code>Or</code></td><td>12005</td><td>12042</td><td>12084</td><td>12169</td><td>12339</td></tr><tr><td><code>Xor</code></td><td>12000</td><td>12000</td><td>12001</td><td>12003</td><td>12006</td></tr><tr><td><code>Add</code></td><td></td><td>12037</td><td>12080</td><td>12167</td><td>12340</td></tr><tr><td><code>Sub</code></td><td></td><td>12080</td><td>12165</td><td>12337</td><td>12679</td></tr><tr><td><code>Mul</code></td><td></td><td>12620</td><td>14571</td><td>22467</td><td>54233</td></tr><tr><td><code>Div</code></td><td></td><td>12969</td><td>15960</td><td>28009</td><td>76377</td></tr><tr><td><code>Rem</code></td><td></td><td>12969</td><td>15960</td><td>28009</td><td>76377</td></tr><tr><td><code>Min</code></td><td>12010</td><td>12121</td><td>12249</td><td>12503</td><td>13012</td></tr><tr><td><code>Max</code></td><td>12010</td><td>12121</td><td>12249</td><td>12503</td><td>13012</td></tr><tr><td><code>Lt</code></td><td>12005</td><td>12080</td><td>12166</td><td>12337</td><td>12679</td></tr><tr><td><code>Gt</code></td><td>12010</td><td>12121</td><td>12249</td><td>12503</td><td>13012</td></tr><tr><td><code>Ge</code></td><td>12005</td><td>12080</td><td>12165</td><td>12337</td><td>12679</td></tr><tr><td><code>Le</code></td><td>12010</td><td>12122</td><td>12249</td><td>12503</td><td>13012</td></tr><tr><td><code>Eq</code></td><td>12000</td><td>12037</td><td>12079</td><td>12164</td><td>12334</td></tr><tr><td><code>Ne</code></td><td>12000</td><td>12037</td><td>12079</td><td>12164</td><td>12334</td></tr><tr><td><code>Shl</code></td><td></td><td>100</td><td>100</td><td>100</td><td>100</td></tr><tr><td><code>Shr</code></td><td></td><td>100</td><td>100</td><td>100</td><td>100</td></tr><tr><td><code>Not</code></td><td>12000</td><td></td><td></td><td></td><td></td></tr></tbody></table>

## Other Special Function and Their Required Gas

{% hint style="info" %}
A more detailed explanation on the functionality of these functions can be found in the following link.
{% endhint %}

{% content-ref url="special-functions/" %}
[special-functions](special-functions/)
{% endcontent-ref %}

<table><thead><tr><th width="262">Operation</th><th>gtBool</th><th>gtUint8</th><th>gtUint16</th><th>gtUint32</th><th>gtUint64</th></tr></thead><tbody><tr><td><code>SetPublic</code></td><td>12000</td><td>12000</td><td>12001</td><td>12003</td><td>12006</td></tr><tr><td><code>Decrypt</code></td><td>12000</td><td>12000</td><td>12001</td><td>12003</td><td>12006</td></tr><tr><td><code>Onboard</code></td><td>47039</td><td>47039</td><td>47039</td><td>47039</td><td>47039</td></tr><tr><td><code>ValidateCiphertext</code></td><td>47039</td><td>47039</td><td>47039</td><td>47039</td><td>47039</td></tr><tr><td><code>Offboard</code></td><td>47039</td><td>47040</td><td>47040</td><td>47042</td><td>47045</td></tr><tr><td><code>OffboardToUser</code></td><td>47039</td><td>47040</td><td>47040</td><td>47042</td><td>47045</td></tr><tr><td><code>Rand</code></td><td>6000</td><td>6000</td><td>6000</td><td>6000</td><td>6000</td></tr><tr><td><code>RandBoundedBits</code></td><td>6000</td><td>6000</td><td>6000</td><td>6000</td><td>6000</td></tr><tr><td><code>Mux</code></td><td>12005</td><td>12041</td><td>12083</td><td>12166</td><td>12332</td></tr><tr><td><code>Transfer</code></td><td></td><td>12201</td><td>12413</td><td>12837</td><td>13685</td></tr><tr><td><code>TransferWithAllowance</code></td><td></td><td>12301</td><td>12619</td><td>13255</td><td>14527</td></tr></tbody></table>
