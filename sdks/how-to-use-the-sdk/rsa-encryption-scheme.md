# RSA encryption scheme

The gcEVM utilizes the RSA encryption scheme to acquire the user AES key necessary for encrypting and decrypting data.

The SDK provides several RSA functionalities to support this.

Below are the function signatures for these functionalities, provided in Python, JavaScript, and Go languages:

* **Generate RSA key pair**

{% tabs %}
{% tab title="Python" %}
```python
def generate_rsa_keypair()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function generateRSAKeyPair() 
```
{% endtab %}

{% tab title="Golang" %}
```go
func GenerateRSAKeyPair() ([]byte, []byte, error)
```
{% endtab %}
{% endtabs %}

* **Encrypt**

{% tabs %}
{% tab title="Python" %}
```python
def encrypt_rsa(public_key_bytes, plaintext)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function encryptRSA(publicKey, plaintext)
```
{% endtab %}

{% tab title="Golang" %}
```go
func EncryptRSA(publicKeyBytes []byte, message []byte) ([]byte, error) 
```
{% endtab %}
{% endtabs %}

* **Decrypt**

{% tabs %}
{% tab title="Python" %}
```python
def decrypt_rsa(private_key_bytes, ciphertext)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function decryptRSA(privateKey, ciphertext)
```
{% endtab %}

{% tab title="Golang" %}
```go
func DecryptRSA(privateKeyBytes []byte, ciphertext []byte) ([]byte, error) 
```
{% endtab %}
{% endtabs %}

#### Example usage - Onboard user

The gcEVM employs AES keys unique to each user for the encryption and decryption of their values.

To retrieve the AES key, a contract is provided that requests the system to return the key associated with the sending user. Further details regarding this process are outlined in the onboard user section:

{% content-ref url="../../core-concepts/accounts/onboard-user.md" %}
[onboard-user.md](../../core-concepts/accounts/onboard-user.md)
{% endcontent-ref %}

The `getUserKey` function in Solidity takes a signed RSA public key as a parameter. It then verifies the signature to ensure the authenticity of the RSA public key. Once the signature is verified, the function proceeds to encrypt the AES key using the verified RSA public key.

For a comprehensive understanding of the sign process, please refer to the detailed explanation provided at:

{% content-ref url="ecdsa-signature.md" %}
[ecdsa-signature.md](ecdsa-signature.md)
{% endcontent-ref %}

We offer a script that accomplishes the following tasks:

1. Generates an RSA key pair.
2. Signs the public key.
3. Invokes the `getUserKey` function in Solidity, passing the signed public key.
4. Accepts the encrypted AES key.
5. Decrypts the AES key using the private RSA key.

Below are examples of such scripts implemented in both Python and JavaScript languages:

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
