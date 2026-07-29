# Getting started on Avalanche Fuji (Day 0)

This page gets you ready to build **Privacy on Avalanche** on **Fuji C-Chain** before you touch PoD Solidity or the TypeScript SDK.

You will:

1. Add **Avalanche Fuji C-Chain** to your wallet  
2. Fund the wallet with **test AVAX**  
3. Configure **Hardhat** or **Foundry** for Fuji  
4. Confirm a transaction on **SnowScan**  
5. Jump into PoD with the right packages and next tutorial  

> **Scope:** This is the **C-Chain** (EVM) testnet — chain ID `43113`. It is not an Avalanche L1/subnet setup guide.

## 1. Network parameters (copy into your wallet)

| Field | Value |
| --- | --- |
| Network name | Avalanche Fuji C-Chain |
| RPC URL | `https://api.avax-test.network/ext/bc/C/rpc` |
| Chain ID | `43113` |
| Currency symbol | AVAX |
| Block explorer | `https://testnet.snowscan.xyz` |

Full PoD contract tables: [Avalanche Fuji](networks/fuji.md).

### Core

[Core](https://core.app/) usually includes Fuji. Switch the network to **Fuji** / **Avalanche Fuji C-Chain**, or add the RPC above if you manage networks manually.

### MetaMask (or any EIP-3085 wallet)

1. Open **Settings → Networks → Add network → Add a network manually**.  
2. Paste the table values above.  
3. Save, then select **Avalanche Fuji C-Chain**.  
4. Confirm the chain ID shows **`43113`** (not Avalanche mainnet `43114`).

## 2. Get test AVAX

PoD two-way requests pay fees in **AVAX** on Fuji (`msg.value`). You need enough for deploy gas **and** Inbox fees.

Official options:

- [Avalanche Core testnet faucet (C-Chain)](https://core.app/tools/testnet-faucet/?subnet=c&token=c)  
- [Avalanche Builder Hub faucet](https://build.avax.network/console/primary-network/faucet)  
- Avalanche support note on faucets: [Is there an AVAX faucet?](https://support.avax.network/en/articles/6110239-is-there-an-avax-faucet)

Faucets may require a mainnet AVAX balance, a Builder Hub account, or a coupon. Request **C-Chain** test AVAX (not only P-Chain).

After funding, your wallet AVAX balance on Fuji should be non-zero.

## 3. Configure your toolchain for Fuji

Use Node.js 18+. Install later when you start coding; for Day 0 you only need a network entry that targets Fuji.

### Hardhat (`hardhat.config.ts` / `.js`)

```ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-ethers";

const FUJI_RPC = process.env.AVALANCHE_FUJI_RPC_URL
  ?? "https://api.avax-test.network/ext/bc/C/rpc";

const config: HardhatUserConfig = {
  solidity: "0.8.26",
  networks: {
    fuji: {
      url: FUJI_RPC,
      chainId: 43113,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
};

export default config;
```

Deploy later with:

```bash
npx hardhat run scripts/deploy.ts --network fuji
```

### Foundry (`foundry.toml`)

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.26"

[rpc_endpoints]
fuji = "${AVALANCHE_FUJI_RPC_URL}"
```

`.env` (example):

```bash
AVALANCHE_FUJI_RPC_URL=https://api.avax-test.network/ext/bc/C/rpc
PRIVATE_KEY=0x...
```

Broadcast later with `--rpc-url fuji` (or the resolved RPC URL) and chain ID `43113`.

### Packages you will install for PoD (not required yet)

When you start the adder tutorial:

```bash
npm install @coti-io/pod-sdk ethers
npm install github:coti-io/coti-contracts#main
```

- **TypeScript:** `@coti-io/pod-sdk`  
- **Solidity:** `@coti-io/coti-contracts` (`PodLib`, `PodUserFuji`)

## 4. Smoke-test on SnowScan

1. From your funded Fuji wallet, send a small amount of AVAX to another address you control (or to yourself).  
2. Open the tx on [SnowScan testnet](https://testnet.snowscan.xyz).  
3. Confirm **status success**, network is Fuji, and the fee was paid in AVAX.

Optional RPC check:

```bash
curl -s -X POST https://api.avax-test.network/ext/bc/C/rpc \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}'
```

Expected result hex: `0xa869` (= decimal **`43113`**).

## 5. PoD context (what comes next)

On Fuji your dApp talks to the shared Inbox:

`0x3b8B70819f27e0438cBcE7f31894f799da52648F`

Private compute still runs on **COTI Testnet** (`7082400`). Solidity preset: **`PodUserFuji`**.

| Next | Link |
| --- | --- |
| Network + contract addresses | [Avalanche Fuji](networks/fuji.md) |
| Fees in AVAX | [How do PoA fees work on Avalanche Fuji?](how-poa-fees-work.md) |
| First dApp | [Tutorial: private Adder on Avalanche Fuji](tutorial-private-adder-fuji.md) |
| Animated Fuji flow | [pod.coti.io/avalanche](https://pod.coti.io/avalanche/index.html) |

## Checklist

- [ ] Wallet on Fuji C-Chain (`43113`)  
- [ ] Test AVAX balance > 0  
- [ ] Hardhat or Foundry has a `fuji` RPC / `chainId: 43113`  
- [ ] At least one successful tx visible on SnowScan  
- [ ] Ready for `@coti-io/pod-sdk` + `@coti-io/coti-contracts`
