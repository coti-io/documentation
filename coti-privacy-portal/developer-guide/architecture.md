# Architecture

The diagram below shows how `PrivateERC20` is composed. It inherits from standard OpenZeppelin contracts for access control and reentrancy protection, and delegates all encrypted arithmetic to the MPC precompile via the `MpcCore` library.

```
PrivateERC20 (abstract)
├── inherits: Context, ERC165, IPrivateERC20, AccessControl, ReentrancyGuard
├── storage:
│   ├── _balances: per-account encrypted balance (MPC ciphertext + user ciphertext)
│   ├── _allowances: mapping(address => mapping(address => Allowance))
│   ├── _totalSupply: ctUint256                    // always decrypts to real supply
│   └── _accountEncryptionAddress: mapping(address => address)
├── roles:
│   ├── DEFAULT_ADMIN_ROLE  → deployer
│   └── MINTER_ROLE         → trusted minters only (see note below)
└── MpcCore (precompile at 0x64)
    ├── setPublic256(uint256) → gtUint256
    ├── validateCiphertext(itUint256) → gtUint256
    ├── transfer(gt, gt, gt) → (gt, gt, gtBool)
    ├── offBoard(gtUint256) → ctUint256
    ├── offBoardToUser(gtUint256, address) → ctUint256
    └── onBoard(ctUint256) → gtUint256
```

### Scope: bridge vs private token layer

These bridges **call** `PrivateERC20` **mint** / **burn** / **transferFrom** with **public** amounts. Failures rooted in **MPC**, **encrypted `burn` / `mint` return paths**, AES, or Snap behavior are triaged under **[PrivateERC20 / MPC docs](privateerc20.sol.md)** and **[burning tokens](burning-tokens.md)** first—not assumed to be a **bridge contract** defect unless public **custody + oracle + allowance** checks already pass.

### Privacy Portal bridge pairs

For **Privacy Portal** production deployments, each **public asset** (native COTI or a specific ERC‑20) has a **dedicated bridge** contract. The matching **private** token should grant **`MINTER_ROLE` only** to **that bridge** (and similarly restricted burn paths the bridge uses), so supply changes for listed assets correspond to custodied public tokens on-chain. Third-party integrations may assign `MINTER_ROLE` to other audited contracts—for Portal-listed tokens, governance should avoid leaving extra minters active.

**Custody edge case (native bridge):** The native bridge’s **native balance** can increase without a user **mint** (for example from a **forced transfer** that does not call `receive` / `deposit`). That **does not** mint private tokens and **does not** increase on-chain **`totalUserLiability`**—it is **not** a deposit. Only the normal **`deposit`** and **`receive`** paths apply fees and mint private COTI.

**ERC‑20 bridges — plain `transfer` is not a deposit:** For standard ERC‑20 assets (e.g. USDC), a user or wallet sending tokens with **`transfer`** only updates the bridge’s **token balance**; the bridge contract **does not execute** `deposit`, so there is **no** private mint, **no** native COTI fee collection, and **no** oracle timestamp binding. **Portal In** (or calling **`deposit`** correctly) is required. Integrators should document this clearly; accidental transfers are **not** supported as implicit deposits. **Token policy:** only **vanilla** (exact balance delta) ERC‑20s are in scope; **`UnexpectedTransferBalance`** remains the on-chain guard—**no** roadmap to support fee-on-transfer or similar in this bridge family.

**Oracle consumer rotation:** `priceOracle` is set **per bridge** (`setPriceOracle`). When a new **`CotiPriceConsumer`** is deployed or replaced, ops should follow a **checklist** and call **`setPriceOracle`** on **every** Privacy Portal bridge (native + each ERC‑20) in a **short window**, so fee quotes and staleness behavior do not **split** across assets. Record the new consumer address with **`maxStaleness`** and each bridge’s **`maxOracleAge`** (see dual freshness runbook).

**Consumer upgrade safety:** Treat **`setPriceOracle` to a new address** like a **sensitive deploy**: **`pause()`** (or otherwise **block user traffic**) until the new consumer’s **bytecode matches the audited / expected artifact** (wrong or malicious consumer can mis-report prices even if Band is healthy). Then **`setPriceOracle`** on all bridges, verify reads, **`unpause()`** when ready.

**Planned maintenance:** For **oracle rotation**, **fee / limit tuning**, or other scheduled work, follow a **published checklist**: **announce** → use **`pause()`** only when a **full stop** is needed, or **`setIsDepositEnabled(false)`** when **blocking new deposits** while keeping **withdrawals** available is enough → apply changes → verify (mirror on **testnet** when possible) → **`unpause`** / re-enable deposits. State clearly which leg is affected so users are not surprised by **`DepositDisabled`** vs **`Pausable: paused`**.

**Audit scope:** External review may treat **`CotiPriceConsumer`** and **Privacy Portal bridges** as **one unit** or as **separate artifacts**—either is fine—but the **same trust story** must be explicit everywhere: **Band `ref` → consumer staleness → bridge `maxOracleAge` + timestamp binding → fee math**. Findings on the consumer affect **every** bridge that points at it.

**`CotiPriceConsumer` staleness tuning:** **`setMaxStaleness`** is **owner-only**. The consumer enforces a **minimum** of **`MIN_STALENESS` (1 hour)**—you cannot set a looser window below that on-chain. That floor is **independent** of each bridge’s **`maxOracleAge`** (which can be shorter, e.g. ~35 minutes); the **stricter** of the two layers still wins for reads that hit both.

**Reading oracle metadata for UIs:** Prefer **`CotiPriceConsumer.getPriceWithMeta`** or the bridge’s **`estimateDepositFee` / `estimateWithdrawFee`** for **`lastUpdated`** and rates—not raw Band **`StdReference.getReferenceData`** from the app. That keeps **consumer staleness** and future policy in **one** place; what you show should match what the bridge will enforce when it calls the consumer.

**Fee math in docs:** **`FEE_DIVISOR` (1e6)** and **basis-point style** parameters are **integrator / developer** details (see Solidity `PrivacyBridge`). The **Privacy Portal** should show **human-readable %** and **fee in COTI** only, without exposing “million divisor” arithmetic to retail users unless they open advanced / developer sections.

**Testnet vs mainnet:** Use the **same** bridge and oracle **behavior** (timestamp binding, dual staleness, native COTI fees, pause/rescue semantics). Only **contract addresses**, **parameter values**, and **Band / RPC** endpoints should differ—so integration tests and support playbooks **transfer** across environments without “testnet-only” shortcuts. **Public docs** should steer **new integrators** to **testnet** first (cheaper mistakes), then **mainnet** after the **checklist** (official addresses, Band symbol check, ERC‑20 **`msg.value`** fee behavior, pause/blacklist reads).

**Integrators (non-Portal):** Third-party apps are **not** required to copy the Privacy Portal UI flow byte-for-byte. They may wrap **`estimateDepositFee` / `estimateWithdrawFee`** (or **`getPriceWithMeta`**) so callers do not hand-type timestamps—SDKs can inject the latest **`cotiLastUpdated` / `tokenLastUpdated`** immediately before `deposit` / `withdraw`. **Native COTI:** plain **`receive()`** remains a **deposit** path **without** oracle row equality binding (fee still uses fresh oracle + staleness). **ERC‑20 today:** on-chain **`deposit` / `withdraw`** still take **two timestamp arguments** and enforce **equality** to the current oracle row; integrators supply them from a fresh read even if their **API** does not expose “oracle timestamp” to end users. A future **opt-out of pin** (e.g. sentinel timestamps, an overload, or an explicit mode flag) is **not** deployed yet—product and audit will choose later; current bytecode requires **pinned** rows for ERC‑20 deposit/withdraw.

**ERC‑20 native fee (`msg.value`):** Integrators should attach **the quoted fee** or a **small buffer** only—enough to satisfy **`_collectDynamicNativeFee`**, not large “just in case” overpay. Excess is **push-refunded** to `msg.sender`, or if that fails, **credited** to **`refundableNativeExcess`** for **`claimRefundableNativeExcess`**. Large overpays create **needless claim** surface and UX risk for smart wallets that reject ETH.

**Reentrancy:** Bridge entrypoints use **`nonReentrant`** (`deposit`, `withdraw`, **`claimRefundableNativeExcess`**, fee sweeps, rescue). The native bridge’s **`receive()`** is also **`nonReentrant`** because it runs full deposit logic—**normal** for a non-trivial `receive`. Integrators must **not** call the same bridge again from **`receive`**, **`fallback`**, or **token hooks** during those calls—including **claiming excess inside a nested bridge call**—or the guard will revert (same class as any DeFi **`nonReentrant`** surface). Use **EOA-style** flows or sequence operations so the bridge is not re-entered mid-tx. Do **not** **`delegatecall`** this implementation from an unaudited proxy pattern; **`ReentrancyGuard`** storage must align with the execution context.

**Bridge `maxOracleAge` changes:** Treat **`setMaxOracleAge`** as **rare and intentional** (e.g. Band cadence or risk policy changes). Whenever it changes, **log it in the same runbook row** as the consumer’s **`maxStaleness`** so support knows **which** freshness check failed (`StaleOracleData` vs `OraclePriceStale`) and two teams do not tune values in isolation.

**Operational monitoring:** Use a **standard** dashboard or alerts (indexer, bot, or observability stack) per bridge: **ERC‑20** — compare **`token.balanceOf(bridge)`** to **`totalUserLiability`**; **native** — compare **`address(bridge).balance`** to **`totalUserLiability` + accumulatedCotiFees`** and account for **`refundableNativeExcess`** credits where relevant. This is **liquidity / bookkeeping** visibility, not a proof of MPC private balances—gaps after **rescue** or stray **`transfer`s** are **ops** signals.

**Support wording:** Do **not** tell users that **`totalUserLiability`** equals “exact private supply on the network.” It only tracks **this bridge’s** mint/burn accounting; **economic** private supply and balances ultimately depend on the **private token + MPC** layer, which can diverge if that layer misbehaved (exceptional). Use the counter for **bridge-side** exposure and dashboards, not as a cryptographic attestation.

**Incident communications:** During outages or investigations, **do not** publicly equate **`totalUserLiability`** with “all user funds are safe” until **ops** has reconciled **custody** (token/native balances on bridges) with obligations and any **migration** state. Prefer factual status: **which flows are paused**, **official channels**, and **next update timing**—not solvency promises from the counter alone.

**Postmortems (internal):** After any material **`pause`**, **`rescue*`**, or custody-impacting event, attach **before/after** snapshots per bridge: **`totalUserLiability`**, **`token.balanceOf(bridge)`** (ERC‑20) or **`address(bridge).balance`** (native), **`accumulatedCotiFees`**, and (where relevant) **`refundableNativeExcess`** / stray **`transfer`** notes—so follow-ups are **data-backed**, not narrative-only.

**Support SLAs:** **`OracleTimestampMismatch`** after an oracle **tick** while the user waited on a wallet confirm is **expected** under strict row binding—**not** a **P1** incident by default. Escalate only if mismatches appear **without** any corresponding **`lastUpdated`** change (possible client, RPC, or indexing bug) or at abnormal volume.

**Deploy provenance (internal, per bridge):** Keep a **release record** for each production deployment: repo **tag / commit**, **artifact or bytecode hash**, deployed **bridge address**, **`feeRecipient` / `rescueRecipient`**, **`priceOracle`** consumer address, **`tokenSymbol`**, **public** and **private** token addresses, and **`grantRole(MINTER_ROLE, …)`** (or equivalent) **tx hashes**. This speeds **audit** follow-up and **incident** reconstruction without relying only on block explorers.

**Band symbol check:** Before mainnet wiring, **verify in staging** (or via script) that Band’s **`getReferenceData(<tokenSymbol>, "USD")`** succeeds and returns sensible **`rate` / `lastUpdatedBase`** for the exact string stored in the bridge. Prevents **typos** in **`tokenSymbol`** that compile but **revert** or mis-price at runtime.

**Blacklist governance:** **`addToBlacklist` / `removeFromBlacklist`** are **owner-only** on-chain; pair them with **off-chain** process: **compliance / legal** sign-off, **written reason**, and **ticket / case id** stored outside the chain so appeals and regulators get **traceability**—avoid ad-hoc toggles without records.

**Ownership handover on bridges:** **`transferOwnership`** revokes **all** `OPERATOR_ROLE` and `DEFAULT_ADMIN_ROLE` members, then grants **only** the new owner both roles. After any handover, **re-grant** `OPERATOR_ROLE` explicitly to **bots / automation** that must call **`setIsDepositEnabled`** or **`setDepositDynamicFee` / `setWithdrawDynamicFee`**—otherwise those jobs **silently stop** until noticed.

**Owner address policy (off-chain):** The contract accepts any non-zero **`newOwner`**, but **governance policy** should use **multisig → multisig** (with **timelock** where practical) and **avoid hot EOAs** as bridge **`owner`**, so pause/rescue/oracle rotation keys match production security expectations. **`renounceOwnership`** is **disabled** (`revert("renounceOwnership disabled")`); vacating governance must go through **`transferOwnership`**, not burning owner in-place.

**Operator access reviews:** On a **fixed cadence** (e.g. quarterly), reconcile **`OPERATOR_ROLE`** on each bridge with the **least-privilege** list in runbooks—remove stale operators, confirm automation addresses, and ensure no drift from “who had the key when we launched.”

**Fee sweep (owner-only, not user withdrawals):** Bridges expose **`withdrawCotiFees`** (ERC‑20 deployments) and **`withdrawFees`** (native bridge) so the **owner** can transfer **`accumulatedCotiFees`** to the immutable **`feeRecipient`**. That is **protocol revenue collection**, not Portal Out / user withdrawal—support and integrators should not conflate it with end-user exits.

**Fee sweep governance:** Do **not** run unattended **sweeper bots** against an **EOA `owner`**—policy is **`owner` = multisig** (or equivalent) so each **`amount`** has **human / quorum** intent. Automation may still **propose** txs to that multisig, but the signing key must not be a **single hot EOA** with sole sweep rights.

**Rescue (emergency only, paused, owner):** **`rescueERC20`** and **`rescueNative`** move custodied assets to the immutable **`rescueRecipient`** while the bridge is **paused**. They are **not** end-user flows; **`totalUserLiability`** is not reduced on rescue—user claims on private supply are handled by **migration / off-chain** process until a new bridge or path makes depositors whole. **`rescueERC20`** cannot target the **private** token address (`CannotRescueBridgeToken`)—rescue is for **public** collateral (or other ERC‑20s mistakenly sent), not for draining private ledger supply via this path.

**Runbooks — stray ERC‑20:** Tokens other than the **bridged public** asset that end up on the bridge balance are recovered with the **same** **`rescueERC20(_token, amount)`** (while **paused**); no separate “dust recovery” contract is required. Choose **`_token`** and **partial `amount`** with the same governance care as rescuing the live collateral.

**Native rescue and `accumulatedCotiFees`:** After **`rescueNative`**, the native bridge **lowers** **`accumulatedCotiFees`** if it would exceed **`address(this).balance`** (native TVL and fee float share one balance). Document for **ops** so post-rescue fee sweeps and dashboards are not misread as a bug—**booked fees** cannot exceed **what is still on the contract**.

### Key Concepts

Understanding  COTI data types is essential before working with any `PrivateERC20` function. Every encrypted operation moves values through these representations in a specific order: user input arrives as `itUint256`, gets loaded into `gtUint256` for in-memory computation, and is stored back on-chain as `ctUint256`.

| Term        | Description                                                                                |
| ----------- | ------------------------------------------------------------------------------------------ |
| `gtUint256` | Garbled-text uint256 — an in-memory encrypted value used during MPC computation            |
| `ctUint256` | Ciphertext uint256 — an encrypted value stored on-chain                                    |
| `itUint256` | Input-text uint256 — an encrypted value submitted by a user (ciphertext + signature)       |
| `MpcCore`   | Solidity library that calls the COTI MPC precompile at `address(0x64)`                     |
| AES Key     | A 16-byte (32 hex char) key derived per wallet during onboarding, used to decrypt balances |

