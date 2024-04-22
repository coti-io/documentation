# 🏋️ Avoid Secret Array indexes

Utilizing encrypted indexes to select an element from an array without disclosing it is inefficient. This approach requires looping over all indexes and comparing for equality secretly, which significantly increases gas costs and should be avoided whenever possible.

<mark style="color:red;">**Avoid**</mark>

```solidity
contract AvoidContract{
    ctUint32 data;
    ctUint32[] secretArray;
    
    function setDataAtSecretIndex(gtUint index) public {
        //...
        gtUint32 gtData=0;
        for (uint32 i = 0; i < secretArray.length; i++) {
            gtAti = MpcCore.onboard(secretArray[i])
            if (i==0){
                gtData = gtAti;
            }
            gtBool isEqual = MpcCore.eq(index, i);
            gtData = MpcCore.mux(isEqual, gtAti , gtData);
        }
        if(gtData!=0) 
            data = MpcCore.offboard(gtData)
    }
}
```

<mark style="color:blue;">**Do**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">:  You should aim for clear indexing when accessing secret arrays.</mark>
