# Onboard Account

Once the account is created and funded with native coin, it needs to be onboarded to the system to obtain an AES key. This AES key may be used for encryption and decryption purposes within the system.

* Script: The script named `onboard_account`, available in both Python and Typescript, is provided to facilitate the the onboarding process.
* Execution: Navigate to the main directory and run the appropriate command depending on the language choice
* Output: After running the script, an AES key is added to the `.env` file.
* Environment Loading: The AES key is loaded via code and used to encrypt and decrypt.

After completing these steps, the user has a functional account with the necessary credentials allowing them to start interacting with the system, including running smart contracts or performing other actions as needed.

You can fund accounts using the COTI [faucet.md](../../readme-1/faucet.md "mention").
