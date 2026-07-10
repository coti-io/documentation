# Snap Integration

{% hint style="info" %}
**Building a React/wagmi dApp?** Use the [COTI Wallet Plugin](../../coti-wallet-plugin/README.md) instead. It handles Snap RPC wiring, AES onboarding, and private balance decryption through a provider + hooks model. This guide is for low-level Snap integration without the wallet plugin.
{% endhint %}

### Integrating your dApp with the COTI MetaMask Snap

If you are building a dApp on the COTI network and want it to interact with the COTI MetaMask snap, follow these steps:

1. Add `@metamask/providers` to your dApp.

```shell
yarn add @metamask/providers
```

2. Create a function to detect if the user has Metamask to use it as a provider. You can guide yourself with [this repo](https://github.com/MetaMask/template-snap-monorepo/blob/main/packages/site/src/utils/metamask.ts).
3. Create a `MetaMaskProvider` in your dapp, which will let us know if COTI AES key manager is installed in the user's wallet. You can guide yourself with [**this repo**](https://github.com/MetaMask/template-snap-monorepo/blob/main/packages/site/src/hooks/MetamaskContext.tsx).
4. Create the `useRequest` hook to interact with Metamask. You can guide yourself with [**this repo**](https://github.com/MetaMask/template-snap-monorepo/blob/main/packages/site/src/hooks/useRequest.ts).
5. Now, create a hook called `useInvokeKeyManager` to invoke the COTI MetaMask Snap.

```
export type InvokeKeyManagerParams = {
  method: string;
  params?: Record<string, unknown>;
};

export const useInvokeKeyManager = (snapId) => {
  const request = useRequest();

  /**
   * Invoke the requested method.
   *
   * @param params - The invoke params.
   * @param params.method - The method name.
   * @param params.params - The method params.
   * @returns The response.
   */
  const invokeKeyManager = async ({ method, params }: InvokeKeyManagerParams) =>
    request({
      method: 'wallet_invokeSnap',
      params: {
        snapId,
        request: params ? { method, params } : { method },
      },
    });

  return invokeKeyManager;
};
```

6. Optional - You can also create a hook that detects if COTI MetaMask Snap is installed or not.

```
export const useMetaMask = () => {
  const { provider, setInstalledSnap, installedSnap } = useMetaMaskContext();
  const request = useRequest();

    /**
   * Get the Snap informations from MetaMask.
   */
  const getSnap = async () => {
    const snaps = (await request({
      method: 'wallet_getSnaps',
    })) as GetSnapsResponse;

    setInstalledSnap(snaps[defaultSnapOrigin] ?? null);
  };

  useEffect(() => {
    const detect = async () => {
      if (provider) {
        await getSnap();
      }
    };

    detect().catch(console.error);
  }, [provider]);

  return { installedSnap, getSnap };
};
```

7. Done! Now if you want to encrypt or decrypt some data from your dApp, you can use something like this:

To encrypt

```
  const handleEncryptClick = async () => {
    const result = await invokeSnap({
      method: 'encrypt',
      params: { value: 'hello' },
    });
    if (result) {
      alert(result);
    }
  };
```

To decrypt

```
  const handleDecryptClick = async () => {
    const result = await invokeSnap({
      method: 'decrypt',
      params: {
        value: JSON.stringify({
          ciphertext: new Uint8Array([
            230, 250, 246, 145, 200, 66, 40, 179, 108, 187, 128, 135, 216, 44,
            32, 48,
          ]),
          r: new Uint8Array([
            67, 194, 73, 74, 131, 182, 125, 200, 112, 210, 211, 145, 192, 148,
            187, 11,
          ]),
        }),
      },
    });
    console.log('result', result);
    if (result) {
      alert(result);
    }
  };
```

Check the [**Metamask documentation**](https://docs.metamask.io/snaps/) for more information.
