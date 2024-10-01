# 🔌 COTI Remix Plugin

The COTI Remix plugin seamlessly integrates with the Remix IDE, enabling developers to deploy and interact with contracts on the COTI network.

{% hint style="info" %}
**NOTE:** The COTI Remix plugin currently works only with COTI Devnet. Support for COTI Testnet will be published in the coming days.
{% endhint %}

{% embed url="https://youtu.be/o59aENKhkAI?si=zckDPCC4vYFoRvRC" %}
Remix Plugin Demo
{% endembed %}

### Installation

1. Head to the Remix website at [**remix.ethereum.org**](https://remix.ethereum.org/)
2. Click the "Plugin manager" button, located on the lower-left part of the screen in Remix <img src="../.gitbook/assets/image (5).png" alt="" data-size="line">
3. The plugins are organized alphabetically. Keep scrolling until you see the COTI plugin and click "**Activate**".\
   ![](<../.gitbook/assets/image (6).png>)
4. On the file manager permission dialog, tick the "**Remember this choice**" box so the plugin doesn't ask every time it runs. Click "**Accept**".\
   ![](<../.gitbook/assets/image (7).png>)
5. The COTI plugin button will appear on the left hand panel, below the Git button <img src="../.gitbook/assets/image (8).png" alt="" data-size="line">. Click on the plugin button.



{% hint style="info" %}
**IMPORTANT:** In order to use the privacy-preserving features of the COTI network with the Remix plugin, your workspace must include both [`MpcCore.sol`](https://github.com/coti-io/confidentiality-contracts/blob/main/contracts/lib/MpcCore.sol) and [`MpcInterface.sol`](https://github.com/coti-io/confidentiality-contracts/blob/main/contracts/lib/MpcInterface.sol) files.
{% endhint %}

### Features

#### 0. Environment

The environment section (located at the bottom of the plugin) allows you to select the environment you wish to use to connect to the COTI network. Two options are available:

1. **Wallet**: This option uses a browser-based wallet (i.e., MetaMask). You will need to have the COTI network added to your wallet. You may add the COTI network by visiting [Chainlist](https://chainlist.org/chain/13068200).
2. **Manual (Devnet)**: This option creates 2 accounts for you in the COTI network.

Both options are tracked in the `coti_wallet_config.json` file, stored at the root of your workspace.

#### 1. Faucet

The Faucet section will display your account balance and provide a link to the [COTI faucet](https://faucet.coti.io/) on Discord. To request funds send a message to the bot using the format: `devnet <address>`, for example:

```
devnet 0xDa28f69CbB6d6072DA4bb10378fB87e367c6dF0D
```

#### 2. Onboard

The Onboard section of the plugin provides an easy way to generate an AES key. This AES key will be used for encryption and decryption purposes within the COTI network.

If the account has already been onboarded and an AES key has already been created, the key will be displayed in this section.

The Onboard action makes use of the [**`AccountOnboard.sol`**](https://github.com/coti-io/confidentiality-contracts/blob/main/contracts/AccountOnboard/AccountOnboard.sol) smart contract via the Typescript SDK [**`onboard.ts`**](https://github.com/coti-io/coti-sdk-typescript/blob/main/src/account/onboard.ts) script.

Once the `Onboard` button is clicked, the plugin will return data related to your AES key. You may clear this data by clicking on the `Clear AES Key` button.

The AES key is stored in the `coti_wallet_config.json` file as `"user_key"`.

#### 3. Compile & Deploy

The "Compile & Deploy" section of the plugin serves a similar purpose to the native Remix "Compile" feature, with the key difference that it allows users to compile contracts that are making use of the privacy-preserving features of the COTI network.

Four sub-sections are offered:

* `Compile` section
  * **Compiler version drop-down**: Allows user to select the desired compiler version
  * `Compile` action button: Compiles the contract using the configured compiler version.
  * **Contract selection drop-down**: Allows selection of previously compiled contracts.
* `Deploy` section
  * **Value** (in wei, gwei, or ether): Allows configuration of custom gas amount when the default estimated gas is not used.
  * `Deploy` action button: deploys the indicated contract with the configured parameters.
* `Load From Address` section
  * `Address` input field: Input field to enter the address of a previously deployed contract.
  * `Load Contract` action button: Loads contract indicated on the `Address` field.

#### 4. Interact

The "Interact" section of the plugin serves a similar purpose to the native Remix "Interact" feature, with the key difference that it allows users to interact with contracts that are making use of the privacy-preserving features of the COTI network.
