# Examples

## Simple operations

```solidity
contract SimpleSecretContract {
    uint256 public storedData;

    function PrecompileCheck(gtUint8 input) public {

        // Set public value to 5
        gtUint8 publicValue = MpcCore.setPublic8(uint8(5));

        // Add the input value to the public value
        gtUint8 sum = MpcCore.add(publicValue, input); 
        
        // Check if the sum is not equal to the original value
        gtBool isDifferent = MpcCore.ne(sum, publicValue);

        // Use a multiplexer based on the comparison result
        gtMux = MpcCore.mux(isDifferent, publicValue, sum);

        // Retrieve the selected value from the multiplexer
        ctUint8 selectedValue = MpcCore.offBoard(gtMux);

        // Store the selected value
        gtUint8 selectedGtUintValue = MpcCore.onBoard(selectedValue);
        
        // Negate the comparison result
        gtBool isSame = MpcCore.not(isDifferent);

        // Decrypt the negated boolean value
        bool isSameDecrypted = MpcCore.decrypt(isSame);

        // Use the decrypted boolean value to choose which data to store
        if (isSameDecrypted) {
            storedData = uint256(MpcCore.decrypt(selectedGtUintValue )); 
        } else {
            storedData = uint256(MpcCore.decrypt(sum));
        }
    }
}
```

## private erc20 balanceOf and Transfer

```solidity
contract PrivateERC20Contract {
    // ... some variables and functions ...

    // The function returns the encrypted account balance utilizing the user's secret key. 
    // Since the balance is initially encrypted internally using the system's AES key, the user cannot access it. 
    // Thus, the balance undergoes re-encryption using the user's secret key. 
    // As a result, the function is not designated as a "view" function.
    function balanceOf() public returns (ctUint64 balance){
        ctUint64 balance = balances[msg.sender];
        // The balance is saved encrypted using the system key. However, to allow the user to access it, the balance needs to be re-encrypted using the user key. 
        // Therefore, we decrypt the balance (onBoard) and then encrypt it again using the user key (offBoardToUser).
        gtUint64 balanceGt = MpcCore.onBoard(balance);
        return MpcCore.offBoardToUser(balanceGt, msg.sender);
    }

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
