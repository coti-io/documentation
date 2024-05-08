# Accounts

Accounts are the central starting point when interacting with the COTI network and using COTI Services. A COTI account is an entity, a distinct object type, stored in the ledger, that holds coins. Accounts can hold the native COTI coin (COTI), custom fungible, and custom non-fungible tokens (NFTs) created on the COTI network.

The COTI native coin COTI is a utility coin primarily used to pay for transactions and gas fees when interacting with the network.&#x20;

You interact with the network by submitting transactions that modify the ledger's state or submitting query requests that read data from the ledger. Most transactions and queries have a transaction fee that is charged in COTI.

## Creating an account

The easiest way to create an account is by running an existing test from one of the COTI SDKs. If your `.env` file does contain an account, one will be automatically created. Run one of the following tests to create an account.

{% tabs %}
{% tab title="Hardhat" %}
Runnning tests will create an account automatically. The account will be saved to the `.env` file and will need to be funded. Use the COTI [faucet.md](../../readme-1/faucet.md "mention") to request devnet/testnet funds.

1.  Install dependencies

    ```
    yarn
    ```
2.  Build and compile contracts

    ```
    yarn build
    ```
3.  Run tests

    ```
    yarn test
    ```

See the [**Hardhat Readme**](https://github.com/coti-io/confidentiality-contracts?tab=readme-ov-file#hardhat-confidential-contracts---usage) for more info.
{% endtab %}

{% tab title="Python SDK" %}
Run the `native_transfer.py` script, it will transfer a small amount to a random address - demonstrating standard native transfer.

It will create a new EOA (you will see your public address in the script output), and an `ACCOUNT_PRIVATE_KEY` will be recorded in the `.env` file.

See the [**Python SDK Readme**](https://github.com/coti-io/coti-sdk-python) for more info.
{% endtab %}

{% tab title="Typescrypt SDK" %}
Runnning tests will create an account automatically. The account will be saved to the `.env` file and will need to be funded. Use the COTI [faucet.md](../../readme-1/faucet.md "mention") to request devnet/testnet funds.

1.  Install dependencies

    ```
    yarn
    ```
2.  Run ERC20 test

    ```
    yarn erc20
    ```

See the [**Typescrypt SDK Readme**](https://github.com/coti-io/coti-sdk-typescript) for more info
{% endtab %}
{% endtabs %}

