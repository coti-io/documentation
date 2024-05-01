# Onboard Account

Once the account is created and funded with native coins, it needs to be onboarded to the system to obtain an AES key. This AES key may be used for encryption and decryption purposes within the system.

* Script: The script named `onboard_user`, available in both Python and JavaScript, is provided to facilitate the the onboarding process.
* Execution: Navigate to the main directory (Devnet) and run the appropriate command depending on the language choice:

{% tabs %}
{% tab title="Python" %}
```bash
python3 -m onboardUser.scripts.python.onboard_user
```
{% endtab %}

{% tab title="JavaScript" %}
```bash
node onboardUser/scripts/js/onboard_user.mjs
```
{% endtab %}
{% endtabs %}

* Output: After running the script, an AES key is added to the .env file.
*   Environment Loading: The user should load the AES key into their environment using the command:

    ```bash
    source .env
    ```

After completing these steps, the user has a functional account with the necessary credentials and tokens, allowing them to start interacting with the system, including running smart contracts or performing other actions as needed.
