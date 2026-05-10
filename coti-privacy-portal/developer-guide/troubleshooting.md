# Troubleshooting

### Common Errors and Fixes

| Error                             | Cause                                         | Fix                                                   |
| --------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| `UnexpectedTransferBalance` (bridge) | **Deposit or withdraw:** token balance delta ≠ requested amount | **Fee-on-transfer**, deflationary, or non-standard ERC‑20 — **not supported**. Policy: **whitelist only** vanilla tokens; no plan to extend this bridge for exotic ERC‑20 behavior. Use a standard asset or a **different** product if you need rebates |
| Accidental **ERC‑20 `transfer`** to bridge | User sent public tokens with a plain `transfer` instead of **`deposit`** | **Not a deposit** — no private mint. Do not rely on this; use Portal In or `deposit` + fee. Recovery may require **ops** (e.g. paused `rescueERC20` to `rescueRecipient`) — not an automatic user refund path in the bridge |
| `CannotRescueBridgeToken` (bridge) | `rescueERC20` was called with the **private** token address | By design — rescue moves **public** collateral, not private supply. Use migration / separate tooling for private-token situations |
| `InvalidFee` / `InvalidFeeConfiguration` (bridge) | Operator `setDepositDynamicFee` / `setWithdrawDynamicFee` with bad params (e.g. % > cap, `maxFee == 0`, `fixedFee > maxFee`) | **Operator tx only** — reverts, no fee config change; fix inputs and retry |
| `renounceOwnership disabled` (bridge) | Tooling or user called **`renounceOwnership()`** on the bridge | **By design** — use **`transferOwnership`** to a successor (e.g. multisig). Not a bug; document for integrators and governance playbooks |
| `InvalidAddress` (bridge)          | **`address(0)`** passed to **`addOperator` / `removeOperator` / addToBlacklist / removeFromBlacklist / setPriceOracle`** (and similar) | **Operator or owner script mistake** — not an end-user path. Fix calldata and retry |
| `InvalidLimitConfiguration` (bridge) | Owner `setLimits` with `min > max` for deposit or withdraw | **Admin tx only** — call reverts, chain state unchanged; fix parameters and retry (useful as an operator sanity check) |
| `DepositBelowMinimum` / `DepositExceedsMaximum` / `WithdrawBelowMinimum` / `WithdrawExceedsMaximum` | Amount outside bridge `setLimits` range | Portal should match **on-chain** min/max; if you integrated off-chain config only, refresh from the live bridge |
| `AddressBlacklisted` (bridge)      | Owner blacklisted `msg.sender` for this bridge | No deposits, withdrawals, or `claimRefundableNativeExcess` until removed — **policy**, not a wallet glitch. Contact support if you believe this is wrong |
| `AmountZero` (native bridge `receive` / `deposit`) | `msg.value == 0` or fee consumes entire deposit so **net** mint would be zero | Do not send **zero** native COTI; for explicit `deposit`, ensure amount **after** protocol fee is **greater than zero**. Portal should not submit `value: 0` |
| `DepositDisabled` (bridge)         | Operator turned off new deposits (`isDepositEnabled == false`) | Deposits only are blocked; **withdrawals** may still work unless also limited or paused. Wait for ops or use another time |
| `Pausable: paused` (OpenZeppelin)  | Owner paused the bridge (`pause()`)          | **All** user deposit/withdraw paths gated off until unpaused |
| `OracleTimestampMismatch` (bridge) | Oracle row updated between quote and inclusion | Call the estimate/view again immediately before broadcasting; retry after oracle tick |
| `OracleTimestampMismatch` (suspect **client / RPC**) | UI `estimate*` shows **older** `lastUpdated` than another RPC’s `eth_call` to the same bridge at the same block | Triage **Portal / wallet RPC**: wrong endpoint, lagging node, or **cached** `eth_call` across blocks—not a bridge logic bug first. Align estimate and submit RPCs; do not cache oracle rows across blocks |
| `PriceOracleNotSet` (bridge)       | `priceOracle` is `address(0)` at read time | Should **not** happen on a correctly deployed production bridge (constructor and `setPriceOracle` forbid zero). Treat as **mis-deployed contract, wrong address in Portal, or broken state** — engineering / config, not end-user error |
| `InvalidOraclePrice` (bridge)      | Oracle returned **rate == 0** for COTI or the bridged asset | **Feed / reference data** issue — same response class as future-dated timestamps: escalate to **oracle ops**, consider **pausing** bridges if all symbols fail; user retries alone will not fix |
| `OracleMaxAgeZeroDisallowed` (bridge) | Owner called **`setMaxOracleAge(0)`** | **By design** — bridge rejects disabling staleness via zero. Use a **large finite** `maxOracleAge` only if a very lenient test policy is intended; production should keep a sane bound |
| `OracleLastUpdatedInFuture` (bridge) | Oracle `lastUpdated` is **after** `block.timestamp` (impossible in normal conditions) | **Do not** treat as user retry spam — likely **Band / reference clock skew** or bad reference data. **Pause** new bridge traffic if widespread; escalate to **oracle / infra** ops |
| `OraclePriceStale` (bridge)        | Oracle `lastUpdated` is older than the bridge’s `maxOracleAge` at execution | Wait for a fresh Band update; if it persists, check feed health and bridge `maxOracleAge` vs consumer staleness (not the same error as timestamp mismatch) |
| `StalenessTooLow` (`CotiPriceConsumer`) | Constructor or **`setMaxStaleness`** with value **below** `MIN_STALENESS` (**1 hour**) | Deploy / script error — consumer **never** accepts staleness windows shorter than **1 hour** on-chain (do not assume you can match the bridge’s shorter `maxOracleAge` here). Use **≥ 1 hour**; align effective policy with bridge `maxOracleAge` in the runbook |
| `StaleOracleData` (`CotiPriceConsumer`) | Band data older than consumer `maxStaleness` | Oracle layer rejects stale reads first; fix feed or relax consumer threshold per ops runbook |
| `InsufficientAccumulatedFees` (owner fee sweep) | Owner `withdrawCotiFees` / `withdrawFees` **amount** exceeds **`accumulatedCotiFees`** | **Protocol accounting only** — not user principal. Sweep **≤** booked fees; check rounding, partial sweeps, or native `rescueNative` cap adjustment vs `accumulatedCotiFees` |
| `EthTransferFailed` (**`rescueNative`**) | Native transfer to **`rescueRecipient`** **returned false** | Same class as fee sweep: **`rescueRecipient` must accept native COTI**. Deploy-time / ops validation; **not** end-user support. If the recipient cannot be fixed, migration plan + **new bridge** with a valid recipient |
| `EthTransferFailed` (owner **fee sweep**) | Native transfer to **`feeRecipient`** in `withdrawCotiFees` / `withdrawFees` **returned false** | **`feeRecipient` must accept native COTI** (EOA or contract with payable `receive` / `fallback`). Misconfiguration at deploy or wrong treasury — **ops / engineering**; immutable recipient means **new bridge deploy** if the address cannot be fixed to accept ETH |
| `EthTransferFailed` (native **withdraw** payout) | `to.call{value: publicAmount}` to the user **returned false** | Usually the **recipient wallet is a contract** that rejects native COTI (no `receive` / `fallback`). **Not** the same as **`InsufficientEthBalance`** (bridge balance). Withdraw from an **EOA** or a contract that accepts ETH |
| `InsufficientEthBalance` (native withdraw) | Bridge contract balance cannot cover the payout (`amount − fee`) | **Not** “add COTI to your wallet” for this revert — bridge TVL/liquidity or ops (e.g. rescue/migration). Escalate to protocol/ops |
| `InsufficientBridgeLiquidity` (ERC20 withdraw) | Bridge’s **public ERC‑20** balance is below the requested withdrawal amount | Custody / liquidity on the bridge contract, not a user private-balance display bug. Escalate to protocol/ops if unexpected |
| `InsufficientCotiFee` (ERC20 bridge) | `msg.value` below the quoted native COTI protocol fee | Send more native COTI with the tx; re-estimate if the oracle row may have moved (no ERC‑20 allowance applies to native) |
| `AmountZero` on **`claimRefundableNativeExcess`** | Caller has **no** credited excess (`refundableNativeExcess[msg.sender] == 0`) | Harmless — user pays **gas** for a no-op. **Check on-chain credit** (or UI) before claiming; low priority for support |
| `EthTransferFailed` on **`claimRefundableNativeExcess`** | `msg.sender.call{value}` for the credited excess **returned false** | Same as native withdraw to a rejecting contract: **caller must accept native COTI**. The bridge **restores** `refundableNativeExcess[msg.sender]` on revert — **retry** from an EOA or wallet that implements **`receive`**. **Blacklisted** callers cannot claim |
| Native refund push failed          | Recipient wallet rejects unsolicited ETH       | Claim credited native via `claimRefundableNativeExcess` from the **same address** (`msg.sender`); wallet must accept the transfer |
| `PublicAmountsDisabled`           | `publicAmountsEnabled` is false               | Use encrypted variants or re-enable via admin         |
| `ERC20SelfTransferNotAllowed`     | `from == to` in transfer                      | Never transfer to yourself                            |
| `ERC20InvalidMetadata`            | Empty name or symbol in constructor           | Pass non-empty strings                                |
| `ERC20: mint failed`              | MPC precompile returned false                 | Check MPC network status; retry                       |
| `AES key mismatch`                | Wrong AES key used to decrypt                 | Re-onboard wallet to get correct key                  |
| `invalid BytesLike value`         | Passing HardhatEthersSigner to `prepareIT256` | Use `new ethers.Wallet(privateKey, provider)` instead |
| `PRIVATE_AES_KEY_TESTNET not set` | Missing env var                               | Add 32 hex char key to `.env` (no `0x` prefix)        |

### Security Notes

* Deploy only on chains where the MPC precompile at `address(0x64)` is trusted.
* `MINTER_ROLE` must only be granted to audited contracts&#x20;
* Encrypted operations (`mintGt`, `burnGt`, `transferGT`) return `gtBool` and do NOT revert on failure — always decrypt and check the return value.
* Self-transfers (`from == to`) are explicitly blocked at the contract level.
* `transferAndCall` is protected by `nonReentrant` but the callback contract must be trusted.
* `totalSupply()` always returns `0` for privacy — do not rely on it for supply accounting.
