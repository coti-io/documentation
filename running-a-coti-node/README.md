# Running a COTI Node

Running a node helps secure and decentralize the COTI network and support the overall ecosystem. While there are some similarities to other L2 networks, COTI’s architecture has its own nuances and requirements that we’ll cover here.

### What is a COTI Node?

In the COTI network, Full Nodes are decentralized, lean clients that play a critical role in maintaining the network’s security, scalability, and overall functionality. Anyone is able to run a Full Node to support the network and optionally gain rewards.

Running a COTI full node, downloads a copy of the COTI blockchain and verifies the validity of every block. Unlike validator nodes in Ethereum, COTI full nodes do not actively participate in consensus nor in block proposal, as that is the job of the COTI sequencer (see [**COTI Architecture**](https://docs.coti.io/coti-documentation/how-coti-works/introduction/coti-architecture))

***

### Why Run a Node?

Running a COTI Node offers several benefits:

* **Network Participation**: Contribute to the decentralization and robustness of the network.
* **Community Support**: Strengthen the ecosystem and help drive adoption of COTI’s technology.
* **Rewards & Incentives**: Earn fees for processing and confirming transactions (licensed nodes only).

***

### Requirements

COTI Full Node software is provided as a docker image.

{% hint style="info" %}
**Disclaimer:** Successfully operating, troubleshooting, and maintaining a node requires technical proficiency. Familiarity with tools such as Linux, Docker, and Git is assumed. Users not familiar with this technology stack should consider assigning their license to an existing node operator to minimize their technical requirements.
{% endhint %}

#### Software

* **Operating System**: the following operating systems have been certified to run the node software:
  * Ubuntu 24.04.x (see [**Linux Requirements for Docker**](https://docs.docker.com/desktop/setup/install/linux/#general-system-requirements)**)**
* **Docker**: version 28.0.1
* **Docker-Compose:** version 2.29.1

#### Hardware

The following hardware specs are required to run a COTI full node:

| Specification    | Minimum | Recommended | Professional |
| ---------------- | ------- | ----------- | ------------ |
| **vCPUs**        | 2       | 4           | 8            |
| **Memory (GiB)** | 8       | 16          | 64           |
| **Storage**      | 100 GB  | 200 GB      | 500 GB       |

In addition to the above hardware, a reliable, high-bandwidth internet connection is recommended.

### Recommended **minimum** hosted configuration

* AWS: m7a.Large (2 vCPUs, 8GB memory)
* OVH: b2-15 (4 vCPUs, 15GB memory)

_**Disclaimer**: The above configuration has been certified on Testnet; higher transaction volumes on Mainnet may require increased specifications._

### Recommended **optimal** hosted configuration

* AWS: r5n.2xlarge (8 vCPUs, 64GB memory)
* OVH: r2-120 (8 vCPUs, 120GB memory)

### Network Configuration

#### Open Ports: Protocol and Purpose

You should open the following ports in your host firewall (e.g., UFW) **and** in your cloud provider’s security groups to permit inbound traffic.\
Be aware that **different ports use different protocols (TCP/UDP)** depending on their purpose.

<table><thead><tr><th width="80">Port</th><th width="104.5">Protocol</th><th width="275">Purpose</th><th>Notes</th></tr></thead><tbody><tr><td>7400</td><td>TCP</td><td>Peer-to-Peer (P2P) Communication</td><td>Data layer used for establishing a connection, exchanging blocks, and synchronizing blockchain data with other nodes.</td></tr><tr><td>7400</td><td>UDP</td><td>Node Discovery (Discv4/Discv5)</td><td>Discovery layer used to quickly find the addresses of other nodes on the network, including the bootnodes and all other peers.</td></tr><tr><td>8545</td><td>TCP</td><td>HTTP-RPC API</td><td>Used for external applications to query chain data and submit transactions over HTTP.</td></tr><tr><td>8546</td><td>TCP</td><td>WebSocket-RPC API</td><td>Used for real-time communication, allowing external applications to receive live updates and subscribe to blockchain events.<br></td></tr></tbody></table>

* **Static IP**: Required to ensure stable RPC access, enabling continuous health monitoring.

***

### Setting up Your Node Environment

COTI full nodes are run using docker. Docker provides a way for everyone to run battle-tested, reliable images, known to work with the network.

#### Prerequisites

<table data-header-hidden><thead><tr><th width="257.44818115234375"></th><th></th></tr></thead><tbody><tr><td><ol><li>Git</li></ol></td><td>See <a href="https://git-scm.com/book/en/v2/Getting-Started-Installing-Git"><strong>git-scm.com/book/en/v2/Getting-Started-Installing-Git</strong></a></td></tr><tr><td><ol start="2"><li>Docker</li></ol></td><td>See <a href="https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository"><strong>docs.docker.com/engine/install/ubuntu</strong></a></td></tr><tr><td><ol start="3"><li>Docker Compose</li></ol></td><td>See <a href="https://docs.docker.com/compose/install/standalone/"><strong>docs.docker.com/compose/install/standalone</strong></a></td></tr></tbody></table>

### Installation Steps

{% hint style="info" %}
The following recommended steps reflect best practices but should be performed carefully, as they may significantly impact the operating system.
{% endhint %}

1. **Recommended steps:**
   1.  Set host name (where `<name>`is your chosen node name)

       \{% code fullWidth="false" %\}

       ```bash
       sudo hostnamectl set-hostname <name>-full-node
       ```

       \{% endcode %\}
   2.  Update package lists

       \{% code fullWidth="false" %\}

       ```bash
       sudo apt update
       sudo apt upgrade
       ```

       \{% endcode %\}
   3.  Reboot system

       \{% code fullWidth="false" %\}

       ```bash
       sudo reboot
       ```

       \{% endcode %\}
   4.  Update OS

       \{% code fullWidth="false" %\}

       ```bash
       sudo do-release-upgrade
       ```

       \{% endcode %\}
2. **Configure Docker**
   *   Add your user to the `docker` group:

       \{% code fullWidth="false" %\}

       ```bash
       sudo usermod -aG docker $( whoami )
       ```

       \{% endcode %\}
   *   Logout and re-login

       \{% code fullWidth="false" %\}

       ```bash
       logout
       ```

       \{% endcode %\}
3.  **Clone the COTI Full Node project**

    \{% code fullWidth="false" %\}

    ```bash
    git clone https://github.com/coti-io/coti-full-node.git
    ```

    \{% endcode %\}
4.  **Checkout the stable release tag (Recommended):**

    * NOTE: Replace `v1.1.4-mainnet` with the actual latest stable tag (e.g., `v1.1.4-discovery` for Testnet).

    ```bash
    git checkout tags/<latest_stable_tag>
    # Example for Mainnet: git checkout tags/v1.1.4-mainnet
    ```
5. **Start Your Node**
   1.  Navigate to the newly created "coti-full-node" directory

       \{% code fullWidth="false" %\}

       ```bash
       cd coti-full-node
       ```

       \{% endcode %\}
   2.  Execute node start script

       ```
       ./start_coti-full-node.sh
       ```
   3.  Once the docker-compose has started the node, liveliness check will be executed

       ```
       ./liveness_coti-full-node.sh
       ```

       \
       Output example:

       ```
       Initial block number: 208539
       Check 1: Block number is 208540
       Block number has progressed. Node is syncing.
       ```

{% hint style="info" %}
If liveliness check passed locally it means that your node is syncing with the other nodes in the network.
{% endhint %}

5.  **To Check Node Logs**

    ```
    docker logs -f coti-full-node
    ```

### Restarting Your Node

To restart your node follow these steps:

1.  Stop your node

    \{% code fullWidth="false" %\}

    ```bash
    ./stop_coti-full-node.sh
    ```

    \{% endcode %\}
2.  Start your node

    \{% code fullWidth="false" %\}

    ```bash
    ./start_coti-full-node.sh
    ```

    \{% endcode %\}

### Node Configuration

If you are running a node without a license, no further configuration of the node is required. Simply ensure you are connected to the network.

### Verifying Node Functionality

* Metrics: Visit [**uptime.coti.io**](https://uptime.coti.io) to track performance and status.

{% hint style="info" %}
Metrics monitoring is not available yet for Testnet .
{% endhint %}

* Node availability is crucial for the smooth operation of the network.\
  \
  To evaluate node availability, COTI leverages a monitoring platform that publishes this data. A node is considered available if it successfully responds to the `eth_blockNumber` request. Using this request ensures the node is actively synchronized with the network and functioning correctly.

### Incentives

The economic structure of COTI nodes is designed to incentivize active participation and ensure the long-term sustainability of the network.

Licensed Full Nodes that maintain a minimum uptime of 98% are eligible to receive **Validation Rewards**, which are distributed every **103-hour Period** (epoch). This uptime requirement underscores the importance of reliability and consistent performance within the network.

For more information on incentives, visit the "Node Economy" section of the [coti-node-ecosystem-litepaper.md](coti-node-ecosystem-litepaper.md "mention").

### Maintenance & Monitoring

1. **Regular Updates**: Keep your node software updated to the latest version. This ensures you receive security patches and new features.
2. **Resource Usage**: Monitor CPU, RAM, and disk space to ensure uninterrupted operation.
3. **Uptime**: Use a process manager (like `systemd`) or Docker auto-restart policies to keep your node running if it crashes unexpectedly.

### Troubleshooting

#### Common Issues:

* Peer Connection Errors: Ensure your ports are open and your firewall allows inbound connections.
* High Resource Usage: Upgrade your hardware or adjust configuration settings to reduce overhead.

Where to Get Help:

* [**COTI Discord**](https://discord.coti.io/)
* [**COTI Support**](mailto:support@coti.io)

### FAQ

1. How many nodes can I run?
   1. There’s no set limit, but each node requires its own resources. Running multiple nodes can help decentralize the network but comes with higher operational costs.
2. Can I run a node on a VPS or cloud platform?
   1. Absolutely. Just ensure the service meets the hardware, OS, and networking requirements.
3. Do I earn more rewards by running a more powerful node?
   1. Generally, consistently high uptime can lead to more consistent rewards, however, the only measure to qualify for rewards is uptime.
4. Is it mandatory to purchase a node license to run a node?
   1. No. A node license simply allows you to earn rewards for helping decentralize the network, however, it is not necessary to run a COTI node.

### Next Steps

Congratulations on setting up your COTI Node! By running a node, you’re contributing to the security and decentralization of the COTI network.

The following related sections may provide helpful information:

* [coti-node-ecosystem-litepaper.md](coti-node-ecosystem-litepaper.md "mention")
* [**Node Ecosystem**](../node-ecosystem/README.md) — the managed product experience at [dev.nodes.coti.io](https://dev.nodes.coti.io), including the guided [installer](../node-ecosystem/installation.md), a [UI walkthrough](../node-ecosystem/ui-guide.md) of the spin-up flow, and a [glossary](../node-ecosystem/glossary.md) of thermal states and warm-up windows.

{% hint style="warning" %}
**Running a node without a valid FQDN means no rewards.** The ecosystem measures uptime by calling your node's JSON-RPC endpoint through your domain name. A node that is fully synced but not publicly reachable over a valid FQDN will not accrue uptime and will not be eligible for rewards. See [Node Ecosystem → Installation](../node-ecosystem/installation.md).
{% endhint %}
