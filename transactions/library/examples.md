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
