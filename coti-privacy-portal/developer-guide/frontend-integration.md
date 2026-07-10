# Frontend Integration

{% hint style="info" %}
**Building a React/wagmi dApp?** Use the [COTI Wallet Plugin](../../build-on-coti/tools/coti-wallet-plugin/README.md) as the recommended integration path. It handles AES onboarding, unlock, balance decryption, and private operations through `PrivacyBridgeProvider` and hooks — without your dApp handling raw AES keys. The manual examples below remain useful for non-React stacks.
{% endhint %}

## Wallet Onboarding (AES Key)

Before any encrypted operation, the user must be onboarded to get their AES key. **COTI MetaMask Snap** provides key management.

```typescript
import { onboard, ONBOARD_CONTRACT_ADDRESS } from '@coti-io/coti-ethers';

async function onboardWallet(provider: ethers.BrowserProvider) {
    const signer = await provider.getSigner();
    const { aesKey } = await onboard(ONBOARD_CONTRACT_ADDRESS, signer);
    // Store aesKey securely — it's needed for all decrypt operations
    return aesKey;
}
```

## Fetching a Private Balance

```typescript
import { ethers } from 'ethers';
import { decryptUint256 } from '@coti-io/coti-sdk-typescript';

const BALANCE_ABI = [
    "function balanceOf(address) view returns (tuple(uint256 ciphertextHigh, uint256 ciphertextLow))"
];

async function fetchPrivateBalance(
    tokenAddress: string,
    userAddress: string,
    aesKey: string,       // 32 hex chars, no 0x prefix
    decimals: number = 18
): Promise<string> {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = await provider.getSigner();

    const contract = new ethers.Contract(tokenAddress, BALANCE_ABI, signer);
    const encryptedBalance = await contract.balanceOf(userAddress);

    const { ciphertextHigh, ciphertextLow } = encryptedBalance;

    // Zero check — uninitialized balance
    if (ciphertextHigh === 0n && ciphertextLow === 0n) {
        return '0.00';
    }

    const decrypted = decryptUint256({ ciphertextHigh, ciphertextLow }, aesKey);
    return ethers.formatUnits(decrypted, decimals);
}
```

## React Hook Example

```typescript
import { useState, useCallback } from 'react';
import { ethers } from 'ethers';
import { decryptUint256 } from '@coti-io/coti-sdk-typescript';

export function usePrivateBalance(tokenAddress: string, decimals = 18) {
    const [balance, setBalance] = useState('0.00');
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<string | null>(null);

    const refresh = useCallback(async (userAddress: string, aesKey: string) => {
        setLoading(true);
        setError(null);
        try {
            const provider = new ethers.BrowserProvider(window.ethereum);
            const signer = await provider.getSigner();
            const contract = new ethers.Contract(tokenAddress, [
                "function balanceOf(address) view returns (tuple(uint256 ciphertextHigh, uint256 ciphertextLow))"
            ], signer);

            const enc = await contract.balanceOf(userAddress);

            if (enc.ciphertextHigh === 0n && enc.ciphertextLow === 0n) {
                setBalance('0.00');
                return;
            }

            const decrypted = decryptUint256(
                { ciphertextHigh: enc.ciphertextHigh, ciphertextLow: enc.ciphertextLow },
                aesKey
            );
            setBalance(ethers.formatUnits(decrypted, decimals));
        } catch (err: any) {
            setError(err.message);
            setBalance('0.00');
        } finally {
            setLoading(false);
        }
    }, [tokenAddress, decimals]);

    return { balance, loading, error, refresh };
}
```

## Encrypting a Value for Transactions (itUint256)

All encrypted write operations (transfer, approve, mint, burn) require an `itUint256` — a ciphertext + signature pair. This is how you build one in the browser without the Snap:

```typescript
import { ethers } from 'ethers';
import { encodeKey, encrypt } from '@coti-io/coti-sdk-typescript';

async function buildItUint256(
    amountWei: bigint,
    aesKeyHex: string,
    contractAddress: string,
    functionSelector: string,  // e.g. ethers.id("transfer(address,((uint256,uint256),bytes))").slice(0,10)
    walletAddress: string,
    signer: ethers.JsonRpcSigner
) {
    const BLOCK_SIZE = 16;
    const CT_SIZE = 32;
    const userAesKey = encodeKey(aesKeyHex);

    function writeBE(buf: Uint8Array, value: bigint) {
        for (let i = buf.length - 1; i >= 0; i--) {
            buf[i] = Number(value & 0xffn);
            value >>= 8n;
        }
    }

    // Encrypt low 128 bits and high 128 bits separately
    const lowBytes = new Uint8Array(BLOCK_SIZE);
    writeBE(lowBytes, amountWei);
    const { ciphertext: ctLow, r: rLow } = encrypt(userAesKey, lowBytes);
    const highBytes = new Uint8Array(BLOCK_SIZE); // zeros for values <= 128 bits
    const { ciphertext: ctHigh, r: rHigh } = encrypt(userAesKey, highBytes);

    const ct = new Uint8Array([...ctHigh, ...rHigh, ...ctLow, ...rLow]);

    const ctHighHex = Array.from(ct.slice(0, CT_SIZE)).map(b => b.toString(16).padStart(2, '0')).join('');
    const ctLowHex = Array.from(ct.slice(CT_SIZE)).map(b => b.toString(16).padStart(2, '0')).join('');
    const ciphertextHigh = BigInt('0x' + ctHighHex);
    const ciphertextLow = BigInt('0x' + ctLowHex);

    // Sign: solidityPacked(sender, contract, selector, ctHigh, ctLow)
    const message = ethers.solidityPacked(
        ['address', 'address', 'bytes4', 'uint256', 'uint256'],
        [walletAddress, contractAddress, functionSelector, ciphertextHigh, ciphertextLow]
    );
    const signature = await signer.signMessage(ethers.getBytes(message));

    return {
        ciphertext: { ciphertextHigh, ciphertextLow },
        signature
    };
}
```

## Sending an Encrypted Transfer

```typescript
async function encryptedTransfer(
    tokenAddress: string,
    toAddress: string,
    amountWei: bigint,
    aesKey: string,
    signer: ethers.JsonRpcSigner
) {
    const TRANSFER_SELECTOR = ethers.id(
        "transfer(address,((uint256,uint256),bytes))"
    ).slice(0, 10);

    const walletAddress = await signer.getAddress();

    const itValue = await buildItUint256(
        amountWei,
        aesKey,
        tokenAddress,
        TRANSFER_SELECTOR,
        walletAddress,
        signer
    );

    const iface = new ethers.Interface([
        "function transfer(address to, tuple(tuple(uint256 ciphertextHigh, uint256 ciphertextLow) ciphertext, bytes signature) value) returns (uint256)"
    ]);

    const calldata = iface.encodeFunctionData("transfer", [
        toAddress,
        [[itValue.ciphertext.ciphertextHigh, itValue.ciphertext.ciphertextLow], itValue.signature]
    ]);

    const txHash = await (window.ethereum as any).request({
        method: 'eth_sendTransaction',
        params: [{
            from: walletAddress,
            to: tokenAddress,
            data: calldata,
            gas: '0xB71B00'  // 12,000,000
        }]
    });

    const provider = new ethers.BrowserProvider(window.ethereum);
    await provider.waitForTransaction(txHash);
    return txHash;
}
```

## Sending an Encrypted Approve

```typescript
async function encryptedApprove(
    tokenAddress: string,
    spenderAddress: string,
    amountWei: bigint,
    aesKey: string,
    signer: ethers.JsonRpcSigner
) {
    const APPROVE_SELECTOR = ethers.id(
        "approve(address,((uint256,uint256),bytes))"
    ).slice(0, 10);

    const walletAddress = await signer.getAddress();

    const itValue = await buildItUint256(
        amountWei,
        aesKey,
        tokenAddress,
        APPROVE_SELECTOR,
        walletAddress,
        signer
    );

    const iface = new ethers.Interface([
        "function approve(address spender, tuple(tuple(uint256 ciphertextHigh, uint256 ciphertextLow) ciphertext, bytes signature) value) returns (bool)"
    ]);

    const calldata = iface.encodeFunctionData("approve", [
        spenderAddress,
        [[itValue.ciphertext.ciphertextHigh, itValue.ciphertext.ciphertextLow], itValue.signature]
    ]);

    const txHash = await (window.ethereum as any).request({
        method: 'eth_sendTransaction',
        params: [{
            from: walletAddress,
            to: tokenAddress,
            data: calldata,
            gas: '0xB71B00'
        }]
    });

    const provider = new ethers.BrowserProvider(window.ethereum);
    await provider.waitForTransaction(txHash);
    return txHash;
}
```
