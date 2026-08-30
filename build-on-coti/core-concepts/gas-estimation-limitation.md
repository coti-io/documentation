---
description: >-
  eth_estimateGas cannot evaluate garbled circuits, so decrypt of a gtBool
  always returns true during simulation.
---

# Gas Estimation Limitation

{% hint style="warning" %}
During `eth_estimateGas` (and other node-side simulations), the gcEVM **does not evaluate garbled circuits**. `MpcCore.decrypt` on a `gtBool` therefore always returns `true` (`1`). Solidity then always takes the `true` branch of any `require` / `if` that uses that plaintext boolean.

That is a limitation of estimation, not of the real transaction. The mined transaction evaluates the circuit and uses the real decrypted value.
{% endhint %}

Two consequences follow:

1. **False reverts.** `estimateGas` can revert even when the real call would succeed, if the `true` branch is a failing `require` or an explicit `revert`.
2. **Wrong gas.** Even when estimation does not revert, it only walks the `true` branch. If the `false` branch costs more (or less), the returned gas is wrong.

The [MPC Core `checkedAdd` / `checkedSub` / `checkedMul` helpers](../tools/contracts-library/mpc-core.md) already work around this. Custom `require(decrypt(...))` code often does not.

## Why

Garbled-circuit evaluation is what actually decrypts a `gtBool`. Simulation cannot run that computation, so the Decrypt precompile returns a dummy `1` instead of the real bit.

Once the value is a plaintext `bool`, the EVM is ordinary: `require`, `if`, `&&`, and `||` all see `true`.

Operations that **stay garbled** (`eq`, `not`, `mux`, `add`, …) are not affected. Their gas is the fixed precompile cost from [Supported Operations on Private Data Types](secure-operations-and-gas.md). The bug starts at `decrypt`.

## Minimal example: `estimateGas` reverts, the transaction would not

```solidity
import "@coti-io/coti-contracts/contracts/utils/mpc/MpcCore.sol";

contract EstimateGasTrap {
    // `flag` is a garbled bit that must be false for the call to succeed
    // (overflow, unauthorized, …).
    //
    // Real tx: reverts only when flag is true.
    // estimateGas: decrypt(flag) is always true → require always fails.
    function proceedIfClear(gtBool flag) public {
        require(MpcCore.decrypt(flag) == false, "flag set");
    }
}
```

The same trap, written other ways:

```solidity
require(!MpcCore.decrypt(flag));

if (MpcCore.decrypt(flag)) {
    revert("flag set");
}
```

Wallets and SDKs that call `estimateGas` before sending will refuse this transaction even when `flag` is actually `false` on chain and the call should succeed.

<mark style="color:red;">**Don't:**</mark> `require` that the decrypted bit is `false`. Estimation always decrypts to `true`, so estimation always reverts.

## Minimal example: estimation succeeds, gas is still wrong

```solidity
function maybeHeavy(gtBool flag) public {
    if (MpcCore.decrypt(flag)) {
        cheap();      // always taken during estimateGas
    } else {
        expensive();  // real txs with flag == false take this path; estimateGas never sees it
    }
}
```

If `flag` is `false` in the real transaction, the node estimated `cheap()` and the transaction can run out of gas. If `flag` is `true`, you overpay. There is no way for estimation to pick the real branch.

## How to write a revert that still estimates

Write the `require` so that a decrypted value of `true` is the **success** path. If the natural predicate is “fail when this bit is set” (overflow, unauthorized, …), invert it **in the garbled domain** with `MpcCore.not`, then decrypt:

<mark style="color:blue;">**Do**</mark>

```solidity
contract EstimateGasSafe {
    function spend(gtBool allowed) public {
        // decrypt(allowed) is always true during estimation → require passes.
        // On chain, decrypt returns the real bit.
        require(MpcCore.decrypt(allowed), "not allowed");
    }

    function checkedThing(gtBool overflowBit, gtUint64 result) public returns (gtUint64) {
        // Same pattern as MpcCore.checkOverflow:
        // invert first, then require the decrypted value to be true.
        require(MpcCore.decrypt(MpcCore.not(overflowBit)) == true, "overflow");
        return result;
    }
}
```

That is why `MpcCore` does **not** write `require(decrypt(overflowBit) == false)`. That form always fails gas estimation. It inverts the overflow bit and requires `true` instead. See [Check Overflow](../guides/best-practices/check-overflow.md).

When you do not need an EVM revert, keep the predicate garbled and select with `mux`. Estimation then charges the `mux` precompile once and does not branch on a dummy bool:

```solidity
gtUint64 out = MpcCore.mux(overflowBit, zero, result);
```

## What wallets and scripts should do

* An `estimateGas` revert on a contract that decrypts a `gtBool` is **not** proof the transaction will revert. Send with an explicit `gasLimit` (a fallback that covers the real path), or fix the contract’s `require` as above.
* A successful `estimateGas` is the cost of the `true` branch only. Add a buffer that covers the more expensive real branch, or avoid plaintext branching after `decrypt`.

## Other decrypted types

The Decrypt precompile returns dummy `1` for numeric garbled types as well. So `require(MpcCore.decrypt(amount) == 100)` fails estimation even when `amount` really is `100`, and `if (MpcCore.decrypt(amount) > 10)` always takes the `true` branch during estimation.

Treat any plaintext value produced by `MpcCore.decrypt` as untrustworthy for simulation: `require` only against `true` / non-zero when you need estimation to succeed, and do not use the dummy integer as if it were the real amount.
