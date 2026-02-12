# COTI Privacy Portal User Guide

Detailed instructions on how to use the COTI Privacy Portal to bridge, manage, and transact with private tokens.

## Getting Started

1. **Connect Your Wallet**
   *   Visit the [COTI Privacy Portal](https://privacy.coti.io).
   *   Click **Connect Wallet**.
   *   Authorize the connection in the MetaMask popup.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FAWGlIYPCGHqFjgZqND9M%252Fconnect_metamask.png%3Falt%3Dmedia%26token%3D1f9c3e2e-cf3a-49bc-95f1-ce89eab3fb97&width=300&dpr=3&quality=100&sign=42341df5&sv=2" alt="Connect MetaMask" style="max-width: 300px; width: auto; max-height: 500px;"><figcaption><p>Connect MetaMask</p></figcaption></figure>

   Once connected, your Public Token balances (like COTI, WETH, and USDT) will appear in the Public Tokens list.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FvOgCnbSsOkPG0siG2sM7%252Fpublic_tokens.png%3Falt%3Dmedia%26token%3D4d0bd948-fe42-46ba-9117-28265d8defdc&width=768&dpr=3&quality=100&sign=daba71c3&sv=2" alt="Public Tokens" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Public Tokens List</p></figcaption></figure>

> 💡 **Tip:** Use the search bar to quickly find a specific token.

## Viewing Private Balances

Your Private Token balances are encrypted and hidden by default. To view them:

2. **Unlock Private Tokens**
   *   Click the **Unlock** button on the Private Tokens card.
   *   MetaMask will prompt you to authorize access to your security key.
   *   Once unlocked, your decrypted private balances will be displayed.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252Fb2r9DCtrGVvWknircNjS%252Funlock_balance.png%3Falt%3Dmedia%26token%3D5a97097d-fdcc-4593-8462-6cd864f557a5&width=768&dpr=3&quality=100&sign=bfdb90b5&sv=2" alt="Unlock Balance" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Unlock Balance</p></figcaption></figure>

> 🔒 **Privacy Note:** The Private Token balances are encrypted on the blockchain. They are decrypted locally in your browser using your personal AES key stored in the COTI Snap. No one else can see your balances.

## Converting Tokens

The Portal supports two conversion flows:

*   **Portal In:** Public → Private (Deposit)
*   **Portal Out:** Private → Public (Withdraw)

### Portal In: Public → Private

Convert your standard public tokens into privacy-preserving private tokens.

3. **Authorize the Conversion (ERC20 Only)**
   If you are depositing ERC20 tokens (not native COTI), you must first approve the Portal to access your tokens:
   *   Select your desired token from the Public Tokens list.
   *   Enter the amount you wish to convert.
   *   Click **Approve**.
   *   Confirm the Spending Cap approval in MetaMask.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FqQg6LMhZ4wQixWLu5dBP%252Fapprove_spending.png%3Falt%3Dmedia%26token%3Dcda14674-65e6-4348-bbab-55eb4f1b4e2d&width=768&dpr=3&quality=100&sign=6ed6f7c6&sv=2" alt="Approve Spending" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Approve Spending</p></figcaption></figure>

   > ℹ️ **Note:** For native COTI, you can skip the approval step.

4. **Convert to Private**
   *   After approval, click **Portal In**.
   *   Confirm the transaction in MetaMask.
   *   Wait for the transaction to be mined.
   *   Your new Private Tokens will appear in the Private Tokens list.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FgOvlh33BGA1pnu2UG5oe%252Fprivate_tokens.png%3Falt%3Dmedia%26token%3D323ec9e4-59da-4960-be09-73755483493b&width=768&dpr=3&quality=100&sign=a133ac20&sv=2" alt="Private Tokens" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Private Tokens</p></figcaption></figure>

### Portal Out: Private → Public

Convert your private tokens back to standard public tokens.

5. **Withdraw Tokens**
   *   Ensure your private balances are unlocked (see [Step 2](#2-unlock-private-tokens)).
   *   Select a token from the Private Tokens list.
   *   Enter the amount you wish to convert.
   *   Click **Portal Out**.
   *   Confirm the transaction in MetaMask.
   *   Your public tokens will be returned to your wallet.

   <figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FiWafPqJvFtkgjfEZgIuf%252Fportal_out.png%3Falt%3Dmedia%26token%3D53923de9-d2b9-42d7-b805-02f4f186ec65&width=768&dpr=3&quality=100&sign=18682531&sv=2" alt="Portal Out" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Portal Out</p></figcaption></figure>

### Adding Private Tokens to MetaMask

To view your private token balances directly within the COTI MetaMask Snap:

1.  In MetaMask, navigate to **Snaps → COTI**.
2.  Select **Import Token**.
3.  Copy the **Contract Address** from the Private Tokens list in the Portal (via the copy icon).
4.  Paste the contract address and confirm.

<figure><img src="https://docs.coti.io/coti-documentation/~gitbook/image?url=https%3A%2F%2F2557786554-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeC83qbrBhITO4kE7kTNB%252Fuploads%252FJVYyo0zpgT982DuOvdL7%252Fsnap_balance.png%3Falt%3Dmedia%26token%3D10c4ff9e-018b-4fb3-b7aa-8736fafa1497&width=768&dpr=3&quality=100&sign=834e91a&sv=2" alt="Snap Balance" style="max-width: 600px; width: auto; max-height: 500px;"><figcaption><p>Snap Balance</p></figcaption></figure>

Your private token will now appear in your COTI Snap wallet view.

## See Also

*   **[Developer Tutorial](developer-guide.md)**: For technical details on creating and integrating private tokens.
