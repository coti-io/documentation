# How to Encrypt Data On-Chain

The following diagram shows a high level workflow to encrypt data on the COTI network, it assumes the following is true:

* User account has been created
* User AES key has been created

<figure><img src="../../.gitbook/assets/writing_data_onchain.jpg" alt=""><figcaption><p>Flow to encrypt data on-chain</p></figcaption></figure>

This is the overview of the process, using the [**data\_on\_chain.py**](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py) example from the [Python SDK](https://github.com/coti-io/coti-sdk-python/tree/main) as a reference:

1. Overall process to send unencrypted value to network, see method [basic\_clear\_encrypt\_decrypt](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py#L172)
2. First step, method [save\_clear\_value\_network\_encrypted\_in\_contract](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py#L229) calls a solidity function and sends it a clear value parameter, The network will encrypt that. &#x20;
   1. This in turn calls [setSomeEncryptedValue](https://github.com/coti-io/confidentiality-contracts/blob/ba4af39da2b02d9d4f8fdd46d3963f0fe9742a85/contracts/examples/DataOnChain.sol#L41) from the Solidity contract. This is a transitory state to make the clear value available publicly by having it encrypted using the network AES key.
3. Then, Ensure the network block has received clear input correctly using method [validate\_block\_has\_tx\_input\_clear\_value](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py#L243). This ensures the relevant block has the clear value sent initially by the user. demonstrating secnario of sending clear value in block but encrypted onchain.
4. Next, in order to get back the value encrypted with the AES key of the user, calling [save\_network\_encrypted\_to\_user\_encrypted\_in\_contract ](https://github.com/coti-io/coti-sdk-python/blob/6ab71858ac77eff916eb6cdf2557300d0b0ef8f1/examples/data\_onchain/data\_on\_chain.py#L172)will call the solidity function of [setUserSomeEncryptedValue ](https://github.com/coti-io/confidentiality-contracts/blob/ba4af39da2b02d9d4f8fdd46d3963f0fe9742a85/contracts/examples/DataOnChain.sol#L55)and by that making it publicly available to read by a view.\
   Demonstrating how to save a network encrypted value by the account that called it (the function).
5. Get back encrypted value by account using [get\_user\_encrypted\_from\_contract](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py#L178) that calls a view method [getUserSomeEncryptedValue](https://github.com/coti-io/confidentiality-contracts/blob/ba4af39da2b02d9d4f8fdd46d3963f0fe9742a85/contracts/examples/DataOnChain.sol#L33)
6. Decrypt value using [decrypt\_value](https://github.com/coti-io/coti-sdk-python/blob/main/examples/data\_onchain/data\_on\_chain.py#L181)

Simple basic flow demonstration of sending a clear value to contract, having it encrypted then reading it back making sure it is what was sent.
