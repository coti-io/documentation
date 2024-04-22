# Decrypt Output

The gcEVM saves secure values by encapsulating them within an Ciphertext object. To get the actual value, the user should decrypt the ciphertext using his AES key.

The SDK offers a function to decrypt ciphertext to a plaintext. The function signature is provided in Python, JavaScript, and Go languages as follows:

{% tabs %}
{% tab title="Python" %}
```python
def decrypt(key, r, ciphertext)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function decrypt(key, r, ciphertext)
```
{% endtab %}

{% tab title="Golang" %}
```go
func Decrypt(key, r, ct []byte) ([]byte, error)
```
{% endtab %}
{% endtabs %}

The function decrypts the Ciphertext (that includes two values - r and AES(r)^plaintext) using the AES key.

**Input parameters:**

* AES key
* r: Random value used in the encryption
* ciphertext: AES(r) ^ plaintext

**Output:**

* The decrypted plaintext

#### Example usage - Private ERC20

When getting the current balance, the ERC20 contract return the encrypted balance. The user should decrypt it to get his actual balance value.

Below are examples demonstrating how to utilize the decrypt function in both Python and JavaScript languages:

my\_CTBalance is the result of the BalanceOf function.

{% tabs %}
{% tab title="Python" %}
```python
def decrypt_value(my_CTBalance, user_key):
    
    # Convert ct to bytes (big-endian)
    byte_array = my_CTBalance.to_bytes(32, byteorder='big')
    
    # Split ct into two 128-bit arrays r and cipher
    cipher = byte_array[:block_size]
    r = byte_array[block_size:]
    
    # Decrypt the cipher
    decrypted_message = decrypt(user_key, r, cipher)
    
    # Print the decrypted cipher
    decrypted_balance = int.from_bytes(decrypted_message, 'big')
    
    return decrypted_balance
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
function decryptValue(myCTBalance, userKey) {
    // Convert CT to bytes
    let ctString = myCTBalance.toString(hexBase);
    let ctArray = Buffer.from(ctString, 'hex');
    
    // Split CT into two 128-bit arrays r and cipher
    const cipher = ctArray.subarray(0, block_size);
    const r = ctArray.subarray(block_size);

    // Decrypt the cipher
    const decryptedMessage = decrypt(userKey, r, cipher);

    // console.log the decrypted cipher
    const decryptedBalance = parseInt(decryptedMessage.toString('hex'), block_size);

    return decryptedBalance;
}
```
{% endtab %}
{% endtabs %}
