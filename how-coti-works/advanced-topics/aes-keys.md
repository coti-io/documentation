# AES Keys

## Acquiring Your AES Key

The gcEVM utilizes AES keys, unique to each user, for encrypting and decrypting their data. To securely retrieve your AES key, the system provides a precompiled contract designed to retrieve the key associated with your account.

To begin, you must generate an RSA key pair, as RSA encryption is used to securely transmit AES keys. Next, sign the generated RSA public key using your account's private key with the ECDSA signing scheme.

After completing these steps, call the `GetUserKey` function on the network's precompiled contract. Pass your RSA public key and its signature as arguments. The precompiled contract will respond (using an event on the blockchain) with your AES key, encrypted in a way that only your RSA private key could decrypt.

---


### High-Level Goal

The fundamental goal is to allow a user to send private data (like their financial balance, a vote, or a health record) to a smart contract, have the contract perform computations on that data, and get a result, all **without revealing the raw data** to the public blockchain or the gcEVM itself.


### Key Components

Before diving into the process, let's define the core elements:

*   **User's Private Input (`value`):** The secret data you want to encrypt. Let's say it's the number `100`.
*   **User's AES Key (`AES_key`):** A secret key that only the user possesses. It is **never** shared with the gcEVM or put on the blockchain. This is the master key for all of the user's private operations.
*   **Random Number (`random_number`):** A number generated for a single, specific transaction. It's often called a "nonce" (number used once). Its purpose is to ensure that encrypting the same input value multiple times results in a different encrypted output each time. This is crucial for security.
*   **Bitwise XOR (`^` or `⊕`):** A simple but powerful logical operation. The key property we care about is that it's reversible:
    *   `Value XOR Key = Encrypted`
    *   `Encrypted XOR Key = Value`
    This makes it perfect for encryption and decryption.

---

## Part 1: Encrypting Inputs (Sending data TO the gcEVM)

This is the process a user follows to prepare their private data to be sent to a smart contract.

Let's follow an example: Alice wants to send the private value `100` to a specific contract function.

#### Step 1: Generate a "One-Time Pad"
The system doesn't directly use your `AES_key` to encrypt the data. Instead, it uses it to create a temporary, disposable "scrambling key," often called a **keystream** or a one-time pad.

*   **Action:** Alice's wallet software generates a cryptographically secure `random_number`. Let's say it's `7Gk9@pX2`.
*   **Action:** Her wallet then encrypts this `random_number` using her secret `AES_key`.
    *   `keystream = AES_encrypt(key: Alice's_AES_key, data: "7Gk9@pX2")`
    *   The result is a seemingly random string of bytes. Let's call it `encrypted_random_number`.

> **Analogy:** Think of your `AES_key` as a secret decoder ring. The `random_number` ("7Gk9@pX2") is a public page number in a codebook. You use your decoder ring on that specific page to get a unique cipher for today's message. Anyone can see the page number, but only you, with the decoder ring, can generate the correct cipher from it.

#### Step 2: Encrypt the Input Value
Now, Alice uses the `keystream` from Step 1 to encrypt her actual input value (`100`).

*   **Action:** Perform a bitwise XOR operation between the input value and the `keystream`.
    *   `encrypted_amount = 100 XOR keystream`

The result, `encrypted_amount`, is now completely scrambled. Without knowing the `keystream`, it's impossible to reverse the XOR operation and get back to `100`. And since the `keystream` was created with Alice's secret `AES_key`, only she can recreate it.

#### Step 3: Generate the Signature
This "signature" is not a cryptographic signature in the typical public-key sense. It's better described as **data binding** or a **contextual commitment**. Its purpose is to prevent an attacker from taking your `encrypted_amount` and submitting it to a different function or a different contract.

*   **Action:** Concatenate (join together) several pieces of contextual information:
    *   Alice's wallet address (`sender_address`)
    *   The smart contract's address (`contract_address`)
    *   The function she is calling (`target_function`)
    *   The `encrypted_amount` she just calculated in Step 2.
*   **Result:** A long string like: `"0xAlice...0xContract...deposit...0xEncrypted..."`

This bundled data proves that this specific encrypted value was intended for this specific operation, initiated by this specific user.

#### Step 4: Assemble and Send the `Inputtext` Object
Finally, Alice's wallet packages everything into the `Inputtext` object and sends it to the gcEVM.

*   `Inputtext = {`
    *   `encrypted_value: encrypted_amount` (from Step 2)
    *   `signature: "0xAlice...0xContract...deposit...0xEncrypted..."` (from Step 3)
*   `}`

The gcEVM receives this. It can check the integrity by looking at the `signature`, but it **cannot** decrypt the `encrypted_amount`. It will now perform its garbled circuit computations on this encrypted value.

### Flowchart: Encrypting Inputs
```+-------------------------------------------------------------------------+
|                        User's Device (Wallet)                           |
|                                                                         |
|  [ Private Input: 100 ]     [ Generate Random Number ]                  |
|             |                            |                              |
|             |                            v                              |
|             |               [ Encrypt with User's AES Key ]             |
|             |                            |                              |
|             |                            v                              |
|             |                  [ Keystream / One-Time Pad ]             |
|             |                           /                               |
|             |                          /                                |
|             +---------------------> { Bitwise XOR } <--------------------+
|                                         |                               |
|                                         v                               |
|                                 [ Encrypted Amount ]                    |
|                                         |                               |
|                                         |                               |
|  [ User Address ]--+                      |                               |
|                    |                      |                               |
|  [ Contract Addr ]-+---> { Concatenate for Signature } <------------------+
|                    |                      |                               |
|  [ Function Name ]-+                      |                               |
|                                         |                               |
|                                         v                               |
|                                   [ Signature ]                         |
|                                                                         |
+-------------------------------------------------------------------------+
       |                 |
       |                 |
       v                 v
+-----------------------------------+
|        Inputtext Object           |
|  {                              } |
|    encrypted_value: ...,        |
|    signature: ...               |
|  {                              } |
+-----------------------------------+
       |
       v
[ Send to gcEVM ]
```
---

## Part 2: Decrypting Outputs (Getting data FROM the gcEVM)

Now, let's say the computation is done. The gcEVM has an encrypted result that it needs to give back to Alice. The gcEVM itself cannot decrypt it.

The gcEVM stores this result in a `Ciphertext` object.

*   `Ciphertext = {`
    *   `encrypted_value: The_Encrypted_Result`
    *   `random_number: The_Same_Random_Number_From_Encryption`
*   `}`

Notice the crucial difference: the `Ciphertext` object contains the **original random number**, not a signature. This is the key that allows the user to unlock the value.

Let's see how Alice decrypts her result.

#### Step 1: Re-generate the "One-Time Pad"
Alice's wallet receives the `Ciphertext` object.

*   **Action:** It extracts the `random_number` from the object. Let's say it's `7Gk9@pX2` (the same one used to encrypt).
*   **Action:** It uses her own secret `AES_key` to encrypt this `random_number` again.
    *   `keystream = AES_encrypt(key: Alice's_AES_key, data: "7Gk9@pX2")`

Because she is using the **exact same key** and the **exact same random number**, she generates the **exact same `keystream`** that was used to create the encrypted result in the first place.

#### Step 2: Decrypt the Output Value
Now she has the `encrypted_value` from the `Ciphertext` object and the `keystream` she just re-created.

*   **Action:** Perform the same bitwise XOR operation.
    *   `decrypted_value = encrypted_value XOR keystream`

Thanks to the reversible nature of XOR, this operation unscrambles the data, and Alice gets her original plaintext value back.

### Flowchart: Decrypting Outputs
```
+-------------------------------------------------------------------------+
|                                  gcEVM                                  |
|                                                                         |
|                          [ Ciphertext Object ]                          |
|                          /                   \                          |
|                         /                     \                         |
|   [ Encrypted Output ] <-----------------------> [ Random Number ]      |
|                                                                         |
+-------------------------------------------------------------------------+
                 |
                 |
                 v
+-------------------------------------------------------------------------+
|                        User's Device (Wallet)                           |
|                                                                         |
|   [ Random Number ]                               [ Encrypted Output ]  |
|           |                                               |             |
|           v                                               |             |
| [ Encrypt with User's AES Key ]                           |             |
|           |                                               |             |
|           v                                               |             |
| [ Keystream / One-Time Pad ]                              |             |
|           |                                               |             |
|           +-----------------> { Bitwise XOR } <-----------+             |
|                                     |                                   |
|                                     v                                   |
|                         [ Decrypted Plaintext Value ]                     |
|                                                                         |
+-------------------------------------------------------------------------+
```


*   **Confidentiality:** The `AES_key` never leaves the user's device. Since the `keystream` used for the XOR operation depends on this key, no one else can generate it to either encrypt or decrypt data.
*   **Integrity:** The `Inputtext` signature binds the encrypted data to a specific context, preventing it from being re-used maliciously in another transaction.
*   **Uniqueness (Replay Protection):** The `random_number` ensures that if you send the same value `100` twice, the final `encrypted_amount` will be different each time. This prevents observers from identifying patterns on the blockchain (e.g., "Oh, that encrypted value looks like the one Alice sent yesterday, she must be making the same payment").




## Network Key

The "network key" is an AES encryption key fragmented using advanced crptographic techniques (e.g., threshold cryptography) so that each node in the network stores only an encrypted or protected portion. No single node or entity can reconstruct or access the entire key. However, through secure multi-party computation, the gcEVM can process data encrypted with the network key and transform it into a usable format for secure on-chain private computations.
