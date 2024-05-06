# 🚀 Quickstart

This quick start guide will help developers get started with deploying smart contracts onto the COTI network. If you are already familiar with Ethereum, EVM-related technologies, smart contracts and web3, follow the steps below to get started.

You should already be familiar with concepts such as **Ethereum**, **EVM**, and **Smart Contracts**.

If you are new to Ethereum and smart contracts, get familiar with these topics by following these introductory guides:

* [Introduction to Ethereum](https://ethereum.org/en/developers/docs/intro-to-ethereum/)
* [Introduction to Smart Contracts](https://ethereum.org/en/developers/docs/smart-contracts/)
* [Ethereum Virtual Machine (EVM)](https://ethereum.org/en/developers/docs/evm/)

{% hint style="info" %}
For questions & support [**join our Discord**](https://discord.com/invite/wfAQfbc3Df)!
{% endhint %}

If you want to get familiar with COTI and the wider privacy topic, get started with the [**Introduction**](<README (1).md>) section.

1. Download a web3 wallet. We recommend Metamask. See [**Install Metamask wallet**](https://metamask.io/download/)**.**
2. Add the Coti network to Metamask
   1. Open Metamask
   2. Click on the network drop-down on the upper left-hand side
   3. Click on "**+Add network**" at the bottom of the prompt
   4. Click on "**Add a network manually**" at the bottom of the screen
   5. Enter the following details for Coti Devnet
      1. Network name: Coti Devnet
      2. New RPC URL: [https://devnet.coti.io](https://devnet.coti.io)
      3. Chain ID: 13068200
      4. Currency symbol: COTI2
      5. Block explorer URL: leave blank
3. Request devnet funds by visiting the COTI faucet
   1. Head to [**https://faucet.coti.io/**](https://faucet.coti.io/)
   2. Send a message to the bot in the following format: \
      `<network>+<your_eoa_address>`\
      \
      For Example:\
      `devnet 0xDA8004D6AB073B9D5549b7D5599D51FF1191C747`\

4. Clone the desired SDK from GitHub.  Follow instructions for desired SDK on its readme
   1. [**Python SDK**](https://github.com/coti-io/coti-sdk-python)
   2. [**Typescript SDK**](https://github.com/coti-io/coti-sdk-typescript)
   3. [**Hardhat** ](https://github.com/coti-io/confidentiality-contracts?tab=readme-ov-file#hardhat-confidential-contracts---usage)[**Development Environment**](https://github.com/coti-io/confidentiality-contracts?tab=readme-ov-file#hardhat-confidential-contracts---usage)
