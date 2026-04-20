# Glossary

Terms you will encounter in the COTI Node Ecosystem web app, the installer, and this documentation. Definitions are authored for node operators; wording may differ from internal field names while preserving the same meaning.

## Node states

### Node status

The operational state of the node process, derived from the node's JSON-RPC `eth_syncing` method and peer presence:

* **Active** — the node is online and fully synced with the tip of the chain.
* **Syncing** — the node is online but still catching up on blocks.
* **Offline** — the node is not currently observed in the peer set.

"Online" in the UI refers to any node that is **Active** or **Syncing**.

### NFT state

The on-chain thermal flag stored on the node's Soulbound NFT:

* **Hot** — the NFT has been minted and is currently marked hot.
* **Cold** — either the NFT has not been minted yet (treated as cold by default), or the NFT has been marked cold after a prolonged outage.

### Thermal status

The combined state shown to operators, derived from node status + NFT state:

| Node status | NFT state | Thermal status |
|---|---|---|
| Active / Syncing | Cold | **Warming up** |
| Active / Syncing | Hot | **Hot** |
| Offline | Hot | **Cooling down** |
| Offline | Cold | **Cold** |

### Warming up

A node is **warming up** when it is online and reachable but has not yet been continuously present long enough for the ecosystem to consider it stable. No NFT has been minted yet. The operator should keep the node online — the warm-up progress bar in `/my-nodes` tracks remaining time.

### Cooling down

A node is **cooling down** when it was previously hot (NFT minted) but has gone offline. If the node comes back online quickly enough, it remains hot. If it stays offline long enough, the NFT flips to cold and the node must warm up again.

### Time to thermal update

Time remaining until the node transitions thermal state — until it becomes **hot** if currently warming up, or until it becomes **cold** if currently cooling down. Not shown when the node is already in a stable state (hot-while-online or cold-while-offline).

## Warm-up & cooldown windows

The thermal state machine is driven by four configuration values maintained by the ecosystem. They are exposed so operators understand the timing of their node's transitions.

### HOT\_WINDOW\_HOURS

The length of the rolling window used to decide whether a node is stable enough to become hot. Defaults to **103 hours** (one epoch).

### HOT\_THRESHOLD\_HOURS

The minimum number of hours *within* `HOT_WINDOW_HOURS` that a node must be continuously present to qualify as hot and receive its Soulbound NFT. Defaults to **72 hours**.

In plain language: *"To become hot, a node must be seen by peers for at least `HOT_THRESHOLD_HOURS` hours during the last `HOT_WINDOW_HOURS` hours."*

### COLD\_WINDOW\_HOURS

The length of the rolling window used to decide whether a node has become unstable enough to be demoted from hot back to cold. Defaults to **206 hours**.

### COLD\_THRESHOLD\_HOURS

The minimum number of hours *within* `COLD_WINDOW_HOURS` that a previously-hot node must be absent to be marked cold. When reached, the node must warm up again before it can return to hot. Defaults to **144 hours**.

In plain language: *"A hot node that stays offline for more than `COLD_THRESHOLD_HOURS` hours in any rolling `COLD_WINDOW_HOURS` window cools back to cold and must redo the warm-up."*

## Identity & ownership

### Node private key

The 64-hex secret that identifies a node on the network. It is generated locally in the browser (or supplied by the operator), written to the node's `nodekey` file by the installer, and **never transmitted to any COTI server**.

### Node address (public key)

The Ethereum-style address derived from the node private key. It is used as the node's identity on-chain and is the wallet that receives the Soulbound Node NFT.

### Wallet address

The wallet connected to the web app (for example via MetaMask). For the per-operator dashboard to show a node, the connected wallet must be the node address — that is, the wallet that owns the Soulbound NFT.

### FQDN (Fully Qualified Domain Name)

The public hostname the operator configures for their node (for example `node1.example.com`). An A record must point the FQDN to the server's public IP before installation.

The FQDN is a **prerequisite for rewards**: the ecosystem probes the node's JSON-RPC endpoint through the FQDN to determine uptime, so a node without a reachable FQDN cannot accrue uptime and therefore cannot earn rewards. See [installation.md](installation.md).

### RPC URL

The public HTTPS endpoint the node exposes after installation, typically `https://<your-fqdn>/rpc`. This is the URL that Better Stack polls to invoke the Node Health Monitor's health check.

### Soulbound Node NFT

A non-transferable NFT minted to the operator's wallet once the node becomes hot. It stores the node's name and image and is the on-chain proof of node ownership. Because it is soulbound, it cannot be transferred between wallets.

## Rewards

### Epoch

A fixed 103-hour reward period. At the end of each epoch, the rewards service evaluates eligibility and credits eligible nodes.

### Eligibility

The set of rules a node + operator must satisfy in an epoch to receive rewards:

* **Uptime (mandatory).** Node uptime for the epoch is at least the configured percentage.
* **Holdings (either threshold is enough).** The operator wallet meets **at least one** of:
  * **USDC** holdings on COTI network ≥ the configured threshold, or
  * **COTI** holdings ≥ the configured threshold (custodial and non-custodial wallets both count).

Whitelisted operators are exempt from the holdings rule and only need to meet the uptime rule.

### Eligible

A node that was found eligible for rewards in at least one epoch. The home-page stats card counts unique nodes that have ever been eligible; the `/my-nodes` per-epoch history marks each epoch with **Eligible** / **Ineligible**.

### Whitelisted

An operator flag that exempts the wallet from the holdings requirement. Uptime is still required. Whitelisting is managed centrally by COTI; most operators will not be whitelisted.

### Blacklisted

An operator flag that prevents a node from being accepted as a peer candidate. Blacklisted nodes do not appear in the ecosystem's dataset and cannot accrue rewards. Blacklisting is used for operational or abuse reasons and is not a state a normal operator encounters.

### Uptime

The percentage of an epoch during which the node was judged healthy by the monitoring stack. An RPC that merely answers is not enough — the health check confirms that the node is actually operating, and only healthy results accrue uptime. Displayed both per-epoch and all-time.

### Rewards (total / last epoch)

* **Total Rewards** — all-time COTI earned by a node across every epoch it was eligible for.
* **Last Epoch Rewards** — COTI earned in the most recently closed epoch.
* **Next Epoch Rewards** — the size of the pool that will be distributed in the upcoming epoch (configured centrally).

Rewards accrue in the on-chain **rewards smart contract** rather than being deposited directly into the operator's wallet. Operators withdraw their balance via the **Claim Now** button in the **My Node** dashboard, or by calling the rewards smart contract directly.

## Other terms you may see

### Live node heartbeats

The home-page visualization showing the count of nodes currently observed by peer discovery. The pulsing tiles are purely decorative; only the counter reflects real data.

### Better Stack

The external uptime-monitoring platform used by the ecosystem to probe every hot node. Operators do not sign up for or configure Better Stack directly — the ecosystem registers monitors automatically after NFT minting.

### Status page

The public Better Stack dashboard that aggregates every hot node's monitor state (up / down). It is the fastest way to see the current health of the whole fleet. URLs are listed in [Networks](README.md#networks).
