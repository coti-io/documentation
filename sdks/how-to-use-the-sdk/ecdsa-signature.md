# ECDSA Signature

The gcEVM employs the ECDSA signature scheme to acquire the user AES key utilized for data encryption and decryption.

The SDK provides a signing functionality to facilitate this process.

Below are the function signatures for signing, provided in Python, JavaScript, and Go languages:

{% tabs %}
{% tab title="Python" %}
```python
def sign(message, key)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function sign(message, key) 
```
{% endtab %}
{% endtabs %}

The function signs the message using the given ECDSA private key.

**Input parameters:**

* message: The message to sign
* key: ECDSA private key

**Output:**

* The generated signature

#### Example usage - Onboard user

As described in the previous page, when executing the onboard\_user script, the user AES key is obtained. During this process, it's necessary to apply a signature to the RSA public key.

Below are examples demonstrating the usage of the sign function in both Python and JavaScript languages:

{% tabs %}
{% tab title="Python" %}
```python
# Generate new RSA key pair
private_key, public_key = generate_rsa_keypair()
# Sign the RSA public key using ECDSA private key
signedEK = sign(public_key, bytes.fromhex(signing_key[2:]))

# Call the getUserKey function to get the encrypted AES key
receipt = soda_helper.call_contract_transaction("onboard_user", "getUserKey", func_args=[public_key, signedEK])
if receipt is None:
    print("Failed to call the transaction function")
    return
encryptedKey = contract.functions.getSavedUserKey().call()

# Decrypt the aes key using the RSA private key
decrypted_aes_key = decrypt_rsa(private_key, encryptedKey)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
// Generate RSA keys and sign the public key using ECDSA private key
const { publicKey, privateKey } = generateRSAKeyPair();
const signedEK = sign(publicKey, Buffer.from(SIGNING_KEY.slice(2), 'hex'));

// Call the getUserKey function to get the encrypted AES key
await sodaHelper.callContractTransaction("onboard_user", "getUserKey", [publicKey, signedEK]);
const encryptedKey = await sodaHelper.callContractView("onboard_user", "getSavedUserKey")

// Decrypt the AES key using the RSA private key
const buf = Buffer.from(encryptedKey.substring(2), 'hex');
const decryptedAESKey = decryptRSA(privateKey, buf);
```
{% endtab %}
{% endtabs %}
