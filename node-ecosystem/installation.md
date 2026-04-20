# Installation

This page explains what happens when you run the automated installer produced by the [**`/setup`** wizard](ui-guide.md). It is the destination for the **"Learn more about installation"** link in the installer step. See [Networks](README.md#networks) for the testnet and mainnet web-app URLs.

{% hint style="info" %}
This page covers the **automated** installer (the single `curl | sudo bash` command). If you want the **manual** Docker-based procedure (hardware specs, ports, step-by-step Git clone and start scripts, FAQ), see [Running a COTI Node](../running-a-coti-node/README.md).
{% endhint %}

## Certified operating system

The installer is **certified on Ubuntu 24.04 LTS**. It is the only officially tested target.

Other Debian-family distributions may work — the installer will warn and prompt for confirmation — but COTI does not guarantee compatibility and does not support installations on other systems. When in doubt, deploy a fresh Ubuntu 24.04 LTS server.

## What you need before running the installer

1. **A server running Ubuntu 24.04 LTS** with root access.
2. **Free disk space** on the install directory's partition, per network:
   * **Testnet:** at least **100 GB**.
   * **Mainnet:** at least **700 GB** (the mainnet chain history is significantly larger and grows over time).
3. **Ports 80 and 443** free on the server (used for HTTP challenge + HTTPS).
4. **Port 7400 (TCP + UDP)** free and not blocked by firewall or iptables (peer-to-peer + node discovery).
5. **A publicly-resolvable FQDN** (for example `node1.example.com`) with an **A record pointing to the server's public IP** — configured and propagated **before** you run the installer.
6. **A node private key** (64 hex characters) — the wizard can generate one for you, or you can bring your own.

{% hint style="warning" %}
**The FQDN is a reward prerequisite, not just a convenience.** The installer requests a Let's Encrypt certificate for your FQDN, and the ecosystem later probes your node's JSON-RPC **through that FQDN** to measure uptime. If the FQDN is missing, misconfigured, or the certificate cannot be issued, **your node will not be credited with uptime and will not earn rewards** — even if the node is fully synced. See [glossary.md](glossary.md) for the FQDN entry.
{% endhint %}

## The one-line command

The wizard generates a command of this shape:

```bash
curl -sL https://fullnode.<network>.coti.io | sudo bash -s -- "<PRIVATE_KEY>" "<FQDN>"
```

`<network>` is either `mainnet` or `testnet`, `<PRIVATE_KEY>` is the 64-hex node key (with or without the `0x` prefix), and `<FQDN>` is the domain name that already points to your server.

{% hint style="danger" %}
Piping `curl` into `sudo bash` executes a remote script as root. Only run commands you obtained from the official wizard, served over `https://fullnode.mainnet.coti.io` or `https://fullnode.testnet.coti.io`. Inspect the script first if you are unsure — the source is the [`coti-full-node`](https://github.com/coti-io/coti-full-node) repository's `install_coti-full-node.sh`.
{% endhint %}

## What the installer does

The installer is driven by [`install_coti-full-node.sh`](https://github.com/coti-io/coti-full-node/blob/main/install_coti-full-node.sh) in the `coti-full-node` repository. It executes the following phases on your server:

### 1. OS and input validation

* Detects `/etc/os-release` and warns if the distribution is not Ubuntu 24.04.x.
* Confirms the script is running as root.
* Validates that the private key is exactly 64 hex characters.
* Validates the FQDN as a well-formed hostname (≤ 253 chars).

### 2. Environment pre-checks

* Confirms the current directory is writable and has enough free disk space for the selected network (see [What you need](#what-you-need-before-running-the-installer)).
* Confirms ports 80, 443 (for Nginx) and 7400 (for the node) are free.
* Confirms no active `ufw` or `iptables` rules block those ports.

### 3. Host dependencies

Installs the packages required to run the node:

* Docker and Docker Compose (skipped if Docker is already present).
* `certbot` for the Let's Encrypt certificate.
* `curl`, `git`, `jq`, `dnsutils`.

### 4. Repository checkout

Clones the [`coti-full-node`](https://github.com/coti-io/coti-full-node) repository into the current directory on the tagged branch (for example `coti-mainnet`), and refuses to proceed if a clone already exists.

### 5. Node configuration

* Writes a `.env` file with the server's detected public IP and the chosen FQDN.
* Writes the private key into a `nodekey` file for the node process.

### 6. HTTPS and reverse proxy

Unless `--no-nginx` is passed:

* Starts a temporary Nginx on port 80 to serve the ACME challenge.
* Requests a Let's Encrypt certificate for the FQDN via `certbot`.
* Tears the temporary Nginx down.
* Writes the full Nginx config (`/rpc`, `/ws`, `/metrics` upstreams) and starts the production proxy.

This is the step that makes your node reachable at `https://<your-fqdn>/rpc` — which is what the monitoring system uses to check your uptime.

### 7. Node launch

Runs `./start_coti-full-node.sh`, which brings up the COTI full node container under Docker Compose and begins syncing the chain.

## After the command finishes

On success the script prints:

```
 SUCCESS! Your COTI full node is initializing.
 FQDN: https://<your-fqdn>
 Use 'docker logs -f coti-<network>-full-node' to view logs.
```

At this point:

* The node starts syncing blocks from peers.
* The wizard in the browser polls the peer-discovery service every ~60 seconds and advances as soon as your node appears in the peer set.
* You enter the **warm-up period** — once your node has been continuously observed for `HOT_THRESHOLD_HOURS` inside a `HOT_WINDOW_HOURS` window, a Soulbound **Node NFT** is minted to your operator wallet and your node becomes **hot**. See the [Glossary](glossary.md) for the exact definitions.

## Optional flags

The installer accepts two flags that the wizard does not surface:

| Flag | Purpose |
|------|---------|
| `--no-nginx` | Skip the Nginx + Let's Encrypt setup. The node still runs on `:8545` / `:8546` but is **not** publicly reachable over HTTPS. Not suitable for reward-eligible nodes. |
| `--staging` | Use the Let's Encrypt **staging** environment (useful for dry runs; the resulting certificate is not trusted by browsers). |

To use these, invoke the script with the extra argument appended:

```bash
curl -sL https://fullnode.mainnet.coti.io | sudo bash -s -- "0x..." "node1.example.com" --staging
```

## Troubleshooting

* **Certbot failed** — Confirm your A-record has propagated (`dig <your-fqdn>`) and that ports 80/443 are reachable from the public internet. Then re-run the installer.
* **Port already in use** — Stop whatever is occupying the port (common culprits: Apache on 80, an existing Nginx, another COTI install). The installer refuses to continue if 80, 443, or 7400 are busy.
* **`coti-full-node` directory already exists** — The installer requires a clean directory. Remove or archive the previous clone before re-running.
* **Node does not appear in the wizard** — Confirm the node container is running (`docker ps`), the chain is syncing (`docker logs -f coti-<network>-full-node`), and the FQDN resolves to your server's public IP.

For manual node management (restart, stop, logs, cleanup), see [Running a COTI Node → Restarting Your Node](../running-a-coti-node/README.md#restarting-your-node).
