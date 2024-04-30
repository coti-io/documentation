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
4. Clone the desired SDK from GitHub (follow instructions for desired SDK below)

{% tabs %}
{% tab title="Python SDK" %}
1. Open your terminal application of choice (such as iTerm)
2. Navigate to the folder where you want to clone the repo
3. Clone the repository with the following command:
   * `git clone git@github.com:coti-io/coti-sdk-python.git`
   * Navigate to the newly created folder `cd coti-sdk-python`
4. Install pre-reqs for Python SDK
   * Install Python
   * Install PIP
   * Install project dependencies, use the following command (assumes python3):
     * `pip3 install -r requirements.txt`
   * NOTE: **Adjust PYTHONPATH if Needed**: If Python can't find the `libs` package, you might need to adjust the `PYTHONPATH` environment variable to include the root directory of the project. You can use the following command to do so (replacing the project path with your own)

```
export PYTHONPATH="/Users/user/projects/coti-sdk-python:$PYTHONPATH"
```

5. Generate an EOA by running the `native_transfer.py` script. You can do so with the following command:

```
python3 examples/basics/native_transfer.py
```

the output should look as follows:

```
So you dont have an account yet, dont worry... lets create one right now!
Creation done!
provider:  https://devnet.coti.io
chain-id:  13068200
latest block:  0xbd227f0435a9bc676648eef66413b05a0927a176c7572b1d543c1381efecb4be
account address: 0xe7Af9060f47068554a50E4DCb995a58Dd0207BF6
account balance:  0 wei ( 0  ether)
account nonce:  0
Traceback (most recent call last):
  File "/Users/daniel/projects/coti-sdk-python/examples/basics/native_transfer.py", line 24, in <module>
    main()
  File "/Users/user/projects/coti-sdk-python/examples/basics/native_transfer.py", line 12, in main
    validate_minimum_balance(web3)  # validate minimum balance
  File "/Users/user/projects/coti-sdk-python/examples/basics/utils.py", line 69, in validate_minimum_balance
    raise Exception("Not enough balance!, head to discord faucet and getsome...")
Exception: Not enough balance!, head to discord faucet and getsome...
```

6. Make note of the "account address" in the output message. This is the address you will need to fund (either via telegram or via transfer from your Metamask wallet).
7. Once the account receives devnet tokens, re-run the `native_transfer.py` script. You can do so with the following command:

```
python3 examples/basics/native_transfer.py
```

&#x20;the output should look as follows:

```
provider:  https://devnet.coti.io
chain-id:  13068200
latest block:  0x2960c2cb8a5a361de83a95f217ab2e53bd69600d141c044eb0ff3d59abdaf5ae
account address: 0xe7Af9060f47068554a50E4DCb995a58Dd0207BF6
account balance:  100000000000000000000 wei ( 100  ether)
account nonce:  0
AttributeDict({'blockHash': HexBytes('0x905233358576d3eb66f21302887e6e98e64affa47013e7adea13451c0f3f8d59'), 'blockNumber': 2558224, 'contractAddress': None, 'cumulativeGasUsed': 21000, 'effectiveGasPrice': 1000000000, 'from': '0xe7Af9060f47068554a50E4DCb995a58Dd0207BF6', 'gasUsed': 21000, 'logs': [], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0x18bA3cD01313B25a2AC63D752663985A5fD1015a', 'transactionHash': HexBytes('0xd30cc7975c14f5565dfdef703b3d4ad56b34454df6aeb6ea9610c0ddff6fe1aa'), 'transactionIndex': 0, 'type': 0})
```

8. Generate Encryption Key by running the `onboard_account.py` script. The script will request the AES encryption key specific for your account and it will log it in the `.env` file. This is required for every action that carries out computation on the COTI v2 network. You can execute this script by running the following command:

```
python3 examples/onboard/onboard_account.py
```

The output should look as follows:

```
rovider:  https://devnet.coti.io
chain-id:  13068200
latest block:  0x3fdc8194785c35cd90c23149080eaa71fa4912d0cc44b59339f344de2fa8375e
account address: 0xe7Af9060f47068554a50E4DCb995a58Dd0207BF6
account balance:  99999978995000000000 wei ( 99.999978995  ether)
account nonce:  1
tx receipt:  AttributeDict({'blockHash': HexBytes('0xfa72acc9021929d03451f7fb09e6c16408e2aa4d56b784587375e73e757e8c84'), 'blockNumber': 2559505, 'contractAddress': None, 'cumulativeGasUsed': 225968, 'effectiveGasPrice': 30000000000, 'from': '0xe7Af9060f47068554a50E4DCb995a58Dd0207BF6', 'gasUsed': 225968, 'logs': [AttributeDict({'address': '0xbFC922C10B03EA5dbC90b98dfc8fb334849ccB78', 'topics': [HexBytes('0xb67504ecfeef0230a06f661ea388c2947b4125a35e918ebff5889e3553c29c04'), HexBytes('0x000000000000000000000000e7af9060f47068554a50e4dcb995a58dd0207bf6')], 'data': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000100a76cf5a428a97b73c4db472ef76877aee78a399d4406e51639461d577a8265a93a9ddc89bdaa6b09057003524ef61a47592adda8f44722ffb18479ccf0efa040c84de76908f076a3d1e2fbff9d27679f2c47dde37e308aa590e926c36651dd06ca61cb319420658429c903fa3a05e9552931b5188c4898bbe6b18fc3666b134fc47fa38a457618fdfcca8bf0adf8956b8845267a63ccc04aedd26465333cdebddd8a2dfc207ff9f4b827aed54b688556413f8b66d2af1b4e5898895270b5f17a4ce436bfd6214ffbd074f49dae92459947260babaa616193db9ff96c35d4f4df0797cab69e3d73e05da9b375c6e136723d7d1fdb3eb397a31a8e9477dd06cbca'), 'blockNumber': 2559505, 'transactionHash': HexBytes('0x6a3f52579e1eed7a9b1890201a303d1713fbab57db5619870ef3f3dcfaa79681'), 'transactionIndex': 0, 'blockHash': HexBytes('0xfa72acc9021929d03451f7fb09e6c16408e2aa4d56b784587375e73e757e8c84'), 'logIndex': 0, 'removed': False})], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000400000000000000000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000008000000000000000040000100000000000000000008000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0xbFC922C10B03EA5dbC90b98dfc8fb334849ccB78', 'transactionHash': HexBytes('0x6a3f52579e1eed7a9b1890201a303d1713fbab57db5619870ef3f3dcfaa79681'), 'transactionIndex': 0, 'type': 0})
(True, 'ACCOUNT_ENCRYPTION_KEY', '4ca1b315ce3664cc3aae4e729f8e4524')
```

9. Check you `.env` file, two new values should appear, namely:
   * `ACCOUNT_PRIVATE_KEY`
   * `ACCOUNT_ENCRYPTION_KEY`
{% endtab %}

{% tab title="Typescrypt SDK" %}

{% endtab %}

{% tab title="Hardhat SDK" %}

{% endtab %}
{% endtabs %}
