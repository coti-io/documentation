# Features

The COTI Node Ecosystem packages node operation into a small number of high-level features. Each one is surfaced in the web app (see [Networks](README.md#networks) for the testnet and mainnet URLs) and backed by one or more of the ecosystem services described in [backend-services.md](backend-services.md).

## 1. Guided installation

A step-by-step wizard at **`/setup`** takes an operator from a fresh Ubuntu server to a running, HTTPS-fronted COTI full node.

* Generates (or accepts) a node private key locally — the key never leaves the browser.
* Validates the node's **FQDN** via a live DNS lookup before continuing.
* Produces a single-line installer command, tailored to the node's key and domain, that the operator runs as root on the target server.
* Watches the peer-discovery network and advances automatically once the node is seen by peers.

See [installation.md](installation.md) for what the installer does on the server, and [ui-guide.md](ui-guide.md) for the wizard walkthrough.

## 2. Live ecosystem view

The home page of the web app exposes the current state of the node network:

* **Live node heartbeats** — a running count of nodes currently observed by peer discovery.
* **Reward-eligible nodes (last epoch)** — how many nodes met the rules in the previous epoch.
* **COTI dropped (last epoch)** — total rewards distributed in the previous epoch.
* **Total COTI earned** — cumulative rewards paid to operators to date.
* **Nodes table** — every node the system knows about, with operator name, total rewards, online status, uptime, and latency.

This view is read-only and does not require a wallet connection.

## 3. Per-operator dashboard

Once an operator connects a wallet that owns a COTI Node NFT, **`/my-nodes`** becomes the per-operator dashboard:

* Current node status (active / syncing / offline) and thermal state (warming up / hot / cooling down / cold).
* **Warm-up progress bar** for nodes that have just been installed and are not yet hot.
* All-time totals: uptime, rewards, latency.
* Per-epoch rewards history with USDC/COTI snapshots, uptime, earned amount, and eligibility status.
* Edit node flow for renaming the node (the name is stored on the Soulbound NFT).

## 4. Eligibility checks

Anyone can preview whether their wallet meets the reward rules before installing a node. For a node to be eligible in a given epoch **both** conditions below must hold:

* **Uptime (mandatory).** The node's per-epoch uptime must be ≥ the configured percentage.
* **Holdings (either threshold is enough).** The operator's wallet must meet **at least one** of the following:
  * **USDC** holdings on COTI network ≥ the configured threshold, **or**
  * **COTI** holdings ≥ the configured threshold (custodial and non-custodial wallets both count).

The **`/eligibility`** page explains the rules in plain language; the home page exposes a quick "Check My Eligibility" button.

## 5. Automatic uptime monitoring

Once a node has been continuously seen by peer discovery for long enough to be considered **hot**, the ecosystem:

1. Mints a Soulbound **Node NFT** to the operator's wallet.
2. Registers the node's RPC URL with **Better Stack** as a monitored endpoint.
3. Runs a multi-signal health check against the node's RPC **through its DNS** to confirm the node is actually operating — simply responding is not enough.
4. Aggregates the resulting uptime per epoch and exposes it in the per-operator dashboard.

{% hint style="warning" %}
**Rewards require a valid DNS.** The ecosystem only measures uptime by calling the node's RPC through the FQDN the operator supplies during setup. A node without a reachable FQDN cannot be monitored and therefore cannot earn rewards — even if it is fully synced on the network.
{% endhint %}

The operator does not interact with Better Stack directly — monitoring is fully automatic. A **public status page** aggregates every hot node's monitor and is available at the URL listed in [Networks](README.md#networks).

## 6. Rewards distribution

Rewards are distributed each **epoch** (103 hours). At the end of every epoch, the rewards service:

1. Reads the node's per-epoch uptime from the monitoring platform.
2. Reads the operator's USDC and COTI holdings at the epoch snapshot.
3. Evaluates the eligibility rules (see [Feature 4](#4-eligibility-checks)).
4. Records each eligible node's reward allocation in the on-chain **rewards smart contract**.

Rewards are **not** auto-deposited to the operator's wallet. Once the contract has been credited, the operator claims the accrued balance either from the **Claim Now** button in the **My Node** dashboard or by calling the rewards smart contract directly from any wallet they control. Unclaimed rewards remain available until claimed.

The per-operator dashboard shows each past epoch with uptime, holdings, earned amount, and an "Eligible / Ineligible" badge. See the [Node Ecosystem Litepaper](../running-a-coti-node/coti-node-ecosystem-litepaper.md) for the economic model.
