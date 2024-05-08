# Validate Input Text

```solidity
function validateCiphertext(itUint8 memory input) internal returns (gtUint8)
```

* The function gets an inputtext containing a ciphertext and a signature.
* First it verifies the signature.
* Then decrypts the cipher text to obtain the clear input.
* Once the input is revealed, the function integrates it into the gcEVM, resulting in a Garbledtext™ value.

## Usage example

Private ERC20 transfer function accepts an private Inputtext amount to transfer.

```solidity
// Transfers the amount of tokens given inside the IT (encrypted and signed value) to address _to
// params: _to: the address to transfer to
//         _itCT: the encrypted value of the amount to transfer
//         _itSignature: the signature of the amount to transfer
//         revealRes: indicates if we should reveal the result of the transfer
// returns: In case revealRes is true, returns the result of the transfer. In case revealRes is false, always returns true
function transfer(address _to, ctUint64 _itCT, bytes calldata _itSignature, bool revealRes) public returns (bool success){
    // Create IT from ciphertext and signature
    itUint64 memory it;
    it.ciphertext = _itCT;
    it.signature = _itSignature;
    // Verify the IT and transfer the value
    gtBool result = contractTransfer(_to, MpcCore.validateCiphertext(it));
    if (revealRes){
        return MpcCore.decrypt(result);
    } else {
        return true;
    }
}
```
