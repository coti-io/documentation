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

## Native Transfer

The following process will help you deploy the [**`native_transfer.py`**](https://github.com/coti-io/coti-sdk-python/blob/main/examples/basics/native\_transfer.py) example from the [COTI Python SDK](https://github.com/coti-io/coti-sdk-python/tree/main). This script will transfer native funds from your wallet account to a random wallet. It will also:

* Create an account
* Create an EOA private key for execution
* Validate minimum balance

{% hint style="info" %}
Ensure your environment meets all the pre-requisites. Visit the [pre-requisites section of the readme](https://github.com/coti-io/coti-sdk-python/tree/main?tab=readme-ov-file#python-sdk-coti-sdk-python---usage).
{% endhint %}

1.  Clone the Python SDK along with its submodules into your desired location

    ```bash
    git clone --recurse-submodules git@github.com:coti-io/coti-sdk-python.git
    ```


2.  Change directory to the newly create one

    ```bash
    cd coti-sdk-python.git
    ```


3.  Install the project's requirements

    ```bash
    python3 -m pip install -r requirements.txt
    ```


4.  Run the `native_transfer.py` script

    ```bash
    python3 examples/basics/native_transfer.py
    ```

    \
    Running the script will automatically create an account and an `ACCOUNT_PRIVATE_KEY` (visible in the `.env` file). The script output will look something like this:



    ```bash
    So you dont have an account yet, dont worry... lets create one right now!
    Creation done!
    provider:  https://devnet.coti.io
    chain-id:  13068200
    latest block:  0xc9ec7259bad015c46a0bef9b0988cac70a62e2abaed7459b5265e425bc5cecb8
    account address: 0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4
    account balance:  0 wei ( 0  ether)
    account nonce:  0
    Traceback (most recent call last):
      File "/Users/user/projects/coti-sdk-python/examples/basics/native_transfer.py", line 24, in <module>
        main()
      File "/Users/user/projects/coti-sdk-python/examples/basics/native_transfer.py", line 12, in main
        validate_minimum_balance(web3)  # validate minimum balance
      File "/Users/user/projects/coti-sdk-python/examples/basics/utils.py", line 69, in validate_minimum_balance
        raise Exception(
    Exception: Not enough balance!, head to discord faucet and getsome...https://discord.com/channels/386571547508473876/1235539223595978752 , ask the BOT:devnet 0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4

    Process finished with exit code 1

    ```

    \
    It is normal to receive the exception `Not enough balance!` on first run. This will be resolved once the account is funded.\

5. Head to the faucet at [**https://faucet.coti.io**](https://faucet.coti.io) to get devnet funds. Send the following message to the bot:\
   \
   `devnet <your_eoa_address>`\
   \
   The bot will reply with the message:\
   \
   `<username> faucet transferred 5 COTIv2 (devnet)` \

6.  Run the `native_transfer.py` script once more

    ```bash
    python3 examples/basics/native_transfer.py
    ```

    \
    The output will be as follows:

    ```bash
    /Users/user/projects/coti-sdk-python/venv/bin/python /Users/user/projects/coti-sdk-python/examples/basics/native_transfer.py 
    provider:  https://devnet.coti.io
    chain-id:  13068200
    latest block:  0x4f5b68d9ef7debc0f86b4fc4c50a81020c8de315d65b4ce12b4372ebedef4f95
    account address: 0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4
    account balance:  10000000000000000000 wei ( 10  ether)
    account nonce:  0
    AttributeDict({'blockHash': HexBytes('0x3e0534655361da10c9ee6454d622609c900e3f552435acc9cc963e370ca1d36b'), 'blockNumber': 3395902, 'contractAddress': None, 'cumulativeGasUsed': 21000, 'effectiveGasPrice': 1000000000, 'from': '0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4', 'gasUsed': 21000, 'logs': [], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0x4A68774D7679e63Ea42599Fe076a899036B3642B', 'transactionHash': HexBytes('0x99ad02f33a146945ac3a671857ab5134965b1f3e78fd53a97710bcdcb99dfee7'), 'transactionIndex': 0, 'type': 0})

    Process finished with exit code 0
    ```

    \
    Now that your account is created and funded, we can now onboard the account to get our new network key.

## On-board Account

The following process will help you deploy the [**`onboard_account.py`**](https://github.com/coti-io/coti-sdk-python/blob/main/examples/onboard/onboard\_account.py) example from the [COTI Python SDK](https://github.com/coti-io/coti-sdk-python/tree/main). This script onboards an EOA into the network. It will also:

* Create an AES key unique for the user
* Use the AES key to encrypt all data sent back to the wallet

This is a mandatory script for any operation executed in a contract that requires encrypt/decrypt actions.

1.  Run the `onboard_account.py` script

    ```bash
    python3 examples/onboard/onboard_account.py
    ```

    \
    Running the script will automatically create an account and an `ACCOUNT_PRIVATE_KEY` (visible in the `.env` file). The script output will look something like this:

    ```bash
    /Users/user/projects/coti-sdk-python/venv/bin/python /Users/user/projects/coti-sdk-python/examples/basics/native_transfer.py 
    provider:  https://devnet.coti.io
    chain-id:  13068200
    latest block:  0x49b06dd8208e0036b866a25bde06ac35b912be14e5ab510f982c024deea15aea
    account address: 0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4
    account balance:  9999978995000000000 wei ( 9.999978995  ether)
    account nonce:  1
    AttributeDict({'blockHash': HexBytes('0x42834b43e5a374242a1529c3dfb31ee086e7e0d0160663e77dfbfae8cf83b711'), 'blockNumber': 3396736, 'contractAddress': None, 'cumulativeGasUsed': 21000, 'effectiveGasPrice': 1000000000, 'from': '0x0287a7A5bD5f4802D4A6048730a11B2713A16bd4', 'gasUsed': 21000, 'logs': [], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0xC9bc0Cb86e176a031D6fFE4A7B063E1A145A6827', 'transactionHash': HexBytes('0x2e5eb95b84835c1dcec2749499233eeb4cc4c536f35d837623aabaf239912ecb'), 'transactionIndex': 0, 'type': 0})

    Process finished with exit code 0
    ```

    \
