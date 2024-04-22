# 📪 Generate an Account

This step involves creating a new user account within the system. The user account will include cryptographic credentials, such as the private key of the account and AES key, which will be used for authentication and secure communication within the system.

#### 1. Generate an account

This step generates a new private key and account.

* Script: The script named gen\_account, available in both Python and JavaScript, is provided to facilitate the generation of user accounts.
* Execution: To execute the script, navigate to the main directory (Devnet) and run the appropriate command depending on the language choice:

{% tabs %}
{% tab title="Python" %}
```bash
python3 -m onboardUser.scripts.python.gen_account
```
{% endtab %}

{% tab title="JavaScript" %}
<pre class="language-bash"><code class="lang-bash"><strong>node onboardUser/scripts/js/gen_account.mjs
</strong></code></pre>
{% endtab %}
{% endtabs %}

* Output: The script will create a new account and save the private key in a .env file in the user's file system.
*   Environment Loading: The user should load the private key into their environment using the command:

    ```bash
    source .env
    ```

After the account was generated, it should be funded with some coins in order to be able to use it to deploy contracts or execute transactions.

This can be done using the faucet, as described in:

{% content-ref url="faucet/" %}
[faucet](faucet/)
{% endcontent-ref %}
