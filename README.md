# 🚀 Quickstart

This quick start guide will help you quickly get started  with Coti. If you are already familiar with Ethereum, EVM-related technologies, smart contracts and web3, follow the steps below to get started.

If you want to get familiar with Coti and the wider privacy topic, get started with the [**Introduction**](<README (1).md>) section.

1. Prerequisites
   1. [**Install Node**](https://nodejs.org/en/learn/getting-started/how-to-install-nodejs)
   2. [**Install Git**](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
   3. [**Install Metamask wallet**](https://metamask.io/download/)
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
3. Request devnet funds: while the Coti faucet becomes operational, request funds via telegram to [`@gmesika`](https://t.me/gmesika) with the following message (replace `<<your_eoa_address>>`with your metamask wallet address)
   * `please send devnet COTI2 to account <<your_eoa_address>>`
4. Clone the desired SDK from GitHub.  Follow instructions for desired SDK on its readme
   1. [**Python SDK**](https://github.com/coti-io/coti-sdk-python)
   2. [**Typescrypt SDK**](https://github.com/coti-io/coti-sdk-typescript)
   3. [**Hardhat SDK**](https://github.com/coti-io/confidentiality-contracts?tab=readme-ov-file#hardhat-confidential-contracts---usage)
