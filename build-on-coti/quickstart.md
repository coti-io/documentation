# Quickstart

This guide will help you explore the basics of interacting with the COTI network and executing transactions with encrypted inputs. To get started, choose your path ([**Solidity**](quickstart.md#solidity), [**TypeScript**](quickstart.md#typescript) or [**Python**](quickstart.md#python)) and follow the steps listed below.

## Solidity

1.  Clone the [**coti-contracts-examples**](https://github.com/coti-io/coti-contracts-examples) repository\


    ```bash
    git clone https://github.com/coti-io/coti-contracts-examples.git
    ```


2.  Navigate to the newly cloned repository directory\


    ```bash
    cd ./coti-contracts-examples
    ```


3.  Install dependencies\


    ```bash
    npm install
    ```


4.  Compile contracts\


    ```bash
    npx hardhat compile
    ```


5.  Run the PrivateToken test script\


    ```bash
    npm run test-private-token
    ```

    \
    Running this test will automatically create an account and a key/value pair with name: `SIGNING_KEYS` (visible in the .env file). The script will output something like this:\


    ```bash
    Private Token
        1) "before all" hook in "Private Token"

      0 passing (39ms)
      1 failing

      1) Confidential ERC20
           "before all" hook in "Private Token":
         Error: Created new random account 0x17EDB982c3569D29EbaF407F72aDD05722d5f179.
         Please use faucet to fund it.
    ```

    \
    It is normal to encounter exception of:\
    `Error: Created new random account [...] Please use faucet to fund it.` \
    on the first run, This will be resolved once the account is funded.\

6. To fund the account, head to the faucet at [**https://faucet.coti.io**](https://faucet.coti.io/) to get Testnet funds. \
   (use [https://discord.coti.io](https://discord.coti.io) to join COTI's Discord server)\
   Send the following message to the faucet along with your newly created account, visible in the last part of the error above.\
   format:\
   `testnet <account address>`\
   for example:\
   `testnet` 0x17EDB982c3569D29EbaF407F72aDD05722d5f179\
   The bot will reply with the message:\
   \
   `<username> faucet transferred 10 COTI (testnet)`\

7.  Run the `PrivateToken` test suite once more.\


    ```bash
    npm run test-private-token
    ```


8.  The script output will look like this:\


    ```bash
    Private Token
    ************* Onboarding user  0x17EDB982c3569D29EbaF407F72aDD05722d5f179  *************
    ************* Onboarding user  0xe1E7315F6970F353661fc84FFd9238133cED3677  *************
    ************* Onboarded! created user key and saved into .env file *************
    ************* Onboarded! created user key and saved into .env file *************
        Deployment
          ✔ Deployed address should not be undefined
          ✔ Owner initial balance (123ms)
          ✔ Function 'name' should be correct (130ms)
          ✔ Function 'symbol' should be correct (123ms)
          ✔ Function 'decimals' should be correct (119ms)
          ✔ Function 'totalSupply' should be correct (117ms)
        Transfer 5
          ✔ Transfer - clear (9469ms)
          ✔ Transfer - Confidential (5260ms)
          ✔ TransferFrom - clear without giving allowance should fail (9905ms)
          ✔ TransferFrom - clear (9770ms)
          ✔ TransferFrom - Confidential (10265ms)
          ✔ Approve/Allowance - Confidential (10255ms)

      12 passing (1m)

    ✨  Done in 69.69s.
    ```



    Running the test suite does the following:

    * **Deploys the `PrivateToken` contract**: Sets up the token with specific details (name, symbol, initial supply).
    * **Tests the deployment**: Verifies the contract address, initial balance, and token details (name, symbol, decimals, total supply).
    * **Tests transfers**: Both clear and confidential transfers, including `transferFrom` functionality with and without prior allowance.
    * **Tests approvals and allowances**: Ensures that the contract correctly handles approvals and allowances, both clear and confidential.


9.  You may also run specific tests:\


    ```
    npm run test-private-nft
    ```

    \
    or\


    ```
    npm run test-private-token
    ```

    \
    or\


    ```
    npm run test-private-auction
    ```

    \
    or\


    ```
    npm run test-private-identity-registry
    ```

    \
    or\


    ```
    npm run test-on-chain-database
    ```

## TypeScript

1.  Clone the [**coti-typescript-examples**](https://github.com/coti-io/coti-typescript-examples) repository\


    ```bash
    git clone https://github.com/coti-io/coti-typescript-examples.git
    ```


2.  Navigate to the [**coti-ethers**](https://github.com/coti-io/coti-typescript-examples/blob/main/coti-ethers/server/README.md) examples subdirectory in the newly cloned repository directory\


    ```bash
    cd ./coti-typescript-examples/coti-ethers/server
    ```


3.  Install dependencies\


    ```bash
    npm install
    ```


4.  Run the `PrivateToken` example\


    ```bash
    npm run erc20
    ```

    \
    Running this test will automatically create an account and a key/value pair with name: `SIGNING_KEY` (visible in the .env file). The script will output something like this:\


    ```
    provider: https://testnet.coti.io/rpc
    chainId: 7082400
    latest block: 1294746
    ************* Created new account  0x1Fc537B022ED68f1Be44d59e4382016d976B3389  and saved into .env file *************
    /Users/spencer/Desktop/COTI/coti-typescript-examples/coti-ethers/server/src/util/general-utils.ts:11
            throw new Error(`Please use faucet to fund account ${wallet.address}`)
                  ^
    Error: Please use faucet to fund account 0x1Fc537B022ED68f1Be44d59e4382016d976B3389
    ```

    \
    It is normal to receive the exception `Error: Please use faucet to fund account` on the first run. This will be resolved once the account is funded.\

5. To fund the account, head to the faucet at [**https://faucet.coti.io**](https://faucet.coti.io/) to get Testnet funds. (use [https://discord.coti.io](https://discord.coti.io) to join COTI's Discord server)\
   Send the following message to the faucet along with your newly created account, visible in the last part of the error above.\
   format:\
   `testnet <account address>`\
   for example:\
   `testnet 0x87c13D0f5903a68bE8288E52b23A220CeC6b1aB6`\
   The bot will reply with the message\
   \
   `<username> faucet transferred 10 COTI (testnet)`\

6.  Run the `PrivateToken` script once more\


    ```bash
    npm run erc20
    ```


7.  You may also run other scripts:\


    ```bash
    npm run nativeTransfer
    ```

    \
    or\


    ```bash
    npm run dataOnChain
    ```

    \
    or\


    ```bash
    npm run onChainDatabase
    ```

## Python

1.  Clone the [**coti-python-examples**](https://github.com/coti-io/coti-python-examples) repository\


    ```bash
    git clone https://github.com/coti-io/coti-python-examples.git
    ```


2.  Navigate to the [**coti-web3**](https://github.com/coti-io/coti-python-examples/blob/main/coti-web3/README.md) examples subdirectory in the newly cloned repository directory\


    ```bash
    cd coti-python-examples/coti-web3
    ```


3.  Install dependencies\


    ```bash
    python3 -m pip install -r requirements.txt
    ```


4.  Set the python path as following\


    ```bash
    export PYTHONPATH=$PWD
    ```


5.  Run the `native_transfer.py` script\


    ```bash
    python3 examples/native_transfer.py
    ```

    \
    Running the script will automatically create an account and a key/value pair with name: `ACCOUNT_PRIVATE_KEY` (visible in the `.env` file). The script will output something like this:\


    ```
    So you dont have an account yet, dont worry... lets create one right now!
    Creation done!
    Traceback (most recent call last):
      File "/Users/user/projects/coti-python-examples/coti-web3/examples/native_transfer.py", line 45, in <module>
        main()
      File "/Users/user/projects/coti-python-examples/coti-web3/examples/native_transfer.py", line 33, in main
        validate_minimum_balance(web3, eoa.address)  # validate minimum balance
      File "/Users/user/projects/coti-python-examples/coti-web3/examples/utils.py", line 43, in validate_minimum_balance
        raise Exception(
    Exception: Not enough balance!, head to discord faucet and get some...https://faucet.coti.io, ask the BOT:testnet 0x78501a70CcADe055D0a1320d640395D505C8Fa97
    ```

    \
    It is normal to receive the exception `Not enough balance!` on the first run. This will be resolved once the account is funded.\

6. To fund the account, head to the faucet at [**https://faucet.coti.io**](https://faucet.coti.io/) to get Testnet funds. (use [https://discord.coti.io](https://discord.coti.io) to join COTI's Discord server)\
   Send the following message to the faucet along with your newly created account, visible in the last part of the error above.\
   format:\
   `testnet <account address>`\
   for example:\
   `testnet 0x87c13D0f5903a68bE8288E52b23A220CeC6b1aB6`\
   The bot will reply with the message\
   \
   `<username> faucet transferred 10 COTI (testnet)`\

7.  Run the `native_transfer.py` script once more\


    ```
    python3 examples/native_transfer.py
    ```

    \
    The script will output as following:\


    ```
    AttributeDict({'blockHash': HexBytes('0xa839081ee841355d2c52548f7fcabda9a665dffa779c87d71251a8b242e45e30'), 'blockNumber': 1294736, 'contractAddress': None, 'cumulativeGasUsed': 63000, 'effectiveGasPrice': 30000000000, 'from': '0x78501a70CcADe055D0a1320d640395D505C8Fa97', 'gasUsed': 21000, 'logs': [], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0x9E4dC0B018200c56dB0d7d17d620F7B551919BdA', 'transactionHash': HexBytes('0xfe348567258fedd45f539218eb145c5b8bda0ef4bc1cf92bbf81363396086823'), 'transactionIndex': 2, 'type': 0})
    ```


8.  Run the `onboard_account.py` script\


    ```
    python3 examples/onboard.py
    ```


9.  You may also run other scripts:\


    ```bash
    python3 examples/confidential_erc20.py
    ```

    \
    or\


    ```bash
    python3 examples/data_on_chain.py
    ```
