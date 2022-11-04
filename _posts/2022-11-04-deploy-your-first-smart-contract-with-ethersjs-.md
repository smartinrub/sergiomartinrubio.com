---
title: Deploy Your First Smart Contract With Ethers.js
image: https://lh3.googleusercontent.com/pw/AL9nZEWIjfI1anNniOVDDidhCDq8RGARA621876bUVWHCk4QjPSibk59WNYN1ERep3BJcpiX1q62ByvL2mr-vlHqxcY4mf882FWWL6NsPvXQnTalO4aGGzhs-MqVESU7X4pfwHyN8ymh9b6eR-svDpK5SJ4D=w2152-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Solidity
    - Blockchain
    - Ethereum
mermaid: false
layout: post
---

We have previously talked about *Blockchain* and *Solidity* on a previous post, so if you don't know what *Blockchain*, *Smart Contracts* or *Solidity* are, go ahead and take a look at [Getting Started with Solidity](https://sergiomartinrubio.com/articles/getting-started-with-solidity/).

The goal is to deploy a *Smart Contract* written in *Solidity* with *JavaScript*!

## Requirements

- Install [NodeJS](https://nodejs.org/en/download/){:target="_blank"}, NPM and/or Yarn.

For MacOS users:

```bash
brew install node
npm install -g corepack
```

>NodeJS comes with NPM and Corepack installs Yarn.

- Install a Solidity compiler for JavaScript. We are going to choose [Solc-JS](https://github.com/ethereum/solc-js){:target="_blank"}.

```bash
yarn add solc
```

You can check the version with `yarn solcjs --version`.

>This would install the compiler for the latest Solidity version. At the time I'm writing this article it's `solc@0.8.17`. You can install a specific version with `yarn add solc@0.8.7-fixed`.

## Compile a Smart Contract

1. Create a folder for your Smart Contracts
2. Create a Smart Contract with Solidity

`MyContract.sol`:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

contract MyContract {
    // content of the contract
}
```

1. Compile Smart Contract

```bash
yarn solcjs --bin --abi --include-path node_modules/ --base-path . -o . MyContract.sol
```

where `--bin` means we want the binaries; `--bin` to generate the Application Binary Interface; `--include-path` to include required libraries; `--base-path` is the path used as the base path; `-o` is where to put the compiled Smart Contract; finally specify the Solidity file that contains the Smart Contract definition.

Two files would be generated the binary (with `bin` extension) and the ABI (with `abi` extension).

## Spin Up a Local Ethereum Blockchain

You can spin up a local Ethereum blockchain with [Ganache](https://trufflesuite.com/ganache/){:target="_blank"}. Once it's installed on your machine you can copy the RPC Server URL (something like `HTTP://127.0.0.1:7545`) and use it on your Smart Contract.

## Connecting to Ethereum Blockchain: RPC

You could interact with the Ethereum Blockchain directly by using the [Ethereum JSON-RCP specification](https://playground.open-rpc.org/?schemaUrl=https://raw.githubusercontent.com/ethereum/execution-apis/assembled-spec/openrpc.json&uiSchema%5BappBar%5D%5Bui:splitView%5D=false&uiSchema%5BappBar%5D%5Bui:input%5D=false&uiSchema%5BappBar%5D%5Bui:examplesDropdown%5D=false){:target="_blank"}. Luckily there are JavaScript libraries out there like [ethers.js](https://docs.ethers.io/v5/){:target="_blank"} or [web3.js](https://web3js.readthedocs.io/en/v1.8.0/){:target="_blank"} that hides the complexity of making RPC API calls.

We are going to use `ethers.js` for this example.

To install *ethers.js* run:

```bash
yarn add ethers
```

Now you can import `ethers.js` and make the connection on your deployment script:

`deploy.js`:

```js
const ethers = require("ethers");

async function main() {
  // Ganache RPC Server: http://127.0.0.1:7545
  const provider = new ethers.providers.JsonRpcProvider(
    "http://127.0.0.1:7545"
  );
  const wallet = new ethers.Wallet(
    "<ONE_OF_THE_PRIVATE_KEYS_FROM_GANACHE>", // put this in an env variable
    provider
  );
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

>It's highly recommended to not hardcode the private keys on your code, instead use something like `dotenv` to store the private key on an `.env` file and read it from your code. You can install `dotenv` with `yarn add dotenv`. From now on we will use env variables. IMPORTANT add `.env` file to your `.gitignore` file. Additionally you can also encrypt the private key.

### Private Key Encryption

We can encrypt the private key with the `ethers.js` library and store the encrypted private key on a file (`.encryptedKey.json`).

`encryptedKey.js`:

```js
const ethers = require("ethers");
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY);
  const encryptedJsonKey = await wallet.encrypt(
    process.env.PRIVATE_KEY_PASSWORD,
    process.env.PRIVATE_KEY
  );
  fs.writeFileSync("./.encryptedKey.json", encryptedJsonKey);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

then we can decrypt the encrypted private key with [fromEncryptedJsonSync()](https://docs.ethers.io/v5/api/signer/#Wallet){:target="_blank"}:

```js
const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
let wallet = new ethers.Wallet.fromEncryptedJsonSync(encryptedJson, process.env.PRIVATE_KEY_PASSWORD);
```

and we can remove from our `.env` file the `PRIVATE_KEY` env variable. 

## Deploy a Smart Contract With ethers.js

Three things are required for deploying the contract to the Ganache Ethereum Blockchain:

1. A connection to the Ganache Server.
2. A wallet private key.
3. The ABI and binary files.

```js
const ethers = require("ethers");
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
  const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
  const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
  let wallet = new ethers.Wallet.fromEncryptedJsonSync(encryptedJson, process.env.PRIVATE_KEY_PASSWORD);
  wallet = await wallet.connect(provider);
  const abi = fs.readFileSync("./MyContract_sol_MyContract.abi", "utf8");
  const binary = fs.readFileSync("./MyContract_sol_MyContract.bin", "utf8");
  const contractFactory = new ethers.ContractFactory(abi, binary, wallet);
  console.log("Start deployment!");
  const contract = await contractFactory.deploy();
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

>It's important to use `await` when we call `deploy()` since the method returns a `Contract` wrapped in a `Promise`.

Once the contract is deployed you can see that some Ethereum have been subtracted on the used address as a result of deploying the contract.

The `contractFactory.deploy()` function accepts optional parameters to override things like the endowment `value`, transaction `nonce`, `gasLimit`, `gasPrice`, `value` or `to`.

Alternatively, you can deploy a contract by providing the transaction details, signing the transaction and finally sending the transaction.

Transaction details:

```js
const nonce = await wallet.getTransactionCount(); // next transaction index 
const tx = {
  nonce: nonce,
  gasPrice: 20000000000, // from Ganache
  gasLimit: 1000000,
  to: null, // we are creating a contract so we are not sending crypto to someone else
  value: 0, // we are creating a contract so we are not sending crypto
  data: "0x<BINARY_CONTENT>", // requires 0x in front of the binary
  chainId: 1337, // network ID on Ganache
};
```

Sign transaction (not required when calling `sendTransaction()`):

```js
const signedTxResponse = await wallet.signTransaction(tx);
```

Send transaction:

```js
const sendTxResponse = await wallet.sendTransaction(tx);
await sendTxResponse.wait(1);
```

`sendTransaction(tx)` signs the transaction and sends it to the network.

>If the deployment fails with `chainId` not found, change the Ganache Network address to 1337.

```js
const ethers = require("ethers");
const fs = require("fs-extra");

async function main() {
  const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
  const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
  let wallet = new ethers.Wallet.fromEncryptedJsonSync(encryptedJson, process.env.PRIVATE_KEY_PASSWORD);
  wallet = await wallet.connect(provider);
  const abi = fs.readFileSync("./MyContract_sol_MyContract.abi", "utf8");
  const binary = fs.readFileSync("./MyContract_sol_MyContract.bin", "utf8");

  const nonce = await wallet.getTransactionCount();
  const tx = {
    nonce: nonce,
    gasPrice: 20000000000,
    gasLimit: 1000000,
    to: null, // we are creating a contract so we are not sending crypto to someone else
    value: 0, // we are creating a contract so we are not sending crypto
    data: "0x<BINARY_CONTENT>",
    chainId: 1337, // network ID on Ganache
  };
  const sendTxResponse = await wallet.sendTransaction(tx);
  await sendTxResponse.wait(1);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### Transaction Receipt

We can get the transaction receipt with:

```js
const transactionReceipt = await contract.deployTransaction.wait(1); // we wait for 1 block confirmation
```

>For getting the transaction receipt you must wait for the block to be confirmed. This is different to the transaction response (`contract.deployTransaction`) which is returned when the transaction is created.

## Interacting With the Smart Contract

The *ABI* (Application Binary Interface) contains all the functions defined in your contract and you can invoke them with *JavaScript*.

Given the following contract:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.8.7;

contract MyContract {
    function helloWorld() public view returns (string memory) {
        return "Hello World!";
    }
}
```

`yarn solcjs --bin --abi --include-path node_modules/ --base-path . -o . MyContract.sol` generates this ABI:

```json
[
   {
      "inputs":[
         
      ],
      "name":"helloWorld",
      "outputs":[
         {
            "internalType":"string",
            "name":"",
            "type":"string"
         }
      ],
      "stateMutability":"view",
      "type":"function"
   }
]
```

so now you can call the function like this:

```js
const ethers = require("ethers");
const fs = require("fs-extra");

async function main() {
  const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
  const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
  let wallet = new ethers.Wallet.fromEncryptedJsonSync(encryptedJson, process.env.PRIVATE_KEY_PASSWORD);
  wallet = await wallet.connect(provider);
  const abi = fs.readFileSync("./MyContract_sol_MyContract.abi", "utf8");
  const binary = fs.readFileSync("./MyContract_sol_MyContract.bin", "utf8");
  const contractFactory = new ethers.ContractFactory(abi, binary, wallet);
  console.log("Deploying contract...");
  const contract = await contractFactory.deploy();
  await contract.deployTransaction.wait(1);

  const helloWorld = await contract.helloWorld();
  console.log(helloWorld);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

`node deploy.js` prints:

```
Deploying contract...
Hello World!
```

## Deploy to a Real Testnet

1. Create account on [Alchemy](https://www.alchemy.com){:target="_blank"}. You can choose a different testnet provider.
2. Create App on Alchemy:
   1. Chain: Ethereum
   2. Network: Goerli (this might change in the future)
3. Click on **View Key** and copy the HTTPS URL.
4. Replace the value of `RPC_URL` with the URL from Alchemy.
5. Get a real Wallet Private Key without real funds (DO NOT USE A PRIVATE KEY WITH REAL FUNDS!).
6. Encrypt the private key: `PRIVATE_KEY=<YOUR_PRIVATE_KEY> PRIVATE_KEY_PASSWORD=<YOUR_PRIVATE_KEY_PASSWORD> node encryptKey.js`
7. Deploy contract: `PRIVATE_KEY_PASSWORD=<YOUR_PRIVATE_KEY_PASSWORD> node deploy.js`
8. Go to the Goerli Etherscan page: https://goerli.etherscan.io and paste your public key. You should be able to see the contract creation! 🎉

>**Alchemy** allows you to use a node to communicate with the blockchain.

## Verify and Publish Smart Contract

1. Go to the contract address (e.g. [https://goerli.etherscan.io/address/0xb7770df4fc2dd064f5867f009ce405ba3a93f5eb](https://goerli.etherscan.io/address/0xb7770df4fc2dd064f5867f009ce405ba3a93f5eb){:target="_blank"})
2. Click on the *Contract* tab and *Verify and Publish*.
3. Select the Compiler Type, Version and License Type that you used on your contract. Finally click on next.
4. Now copy and paste the contract and click on *Verify and Publish*.
5. Now you can click on the *Contract Source Code* to see the contract (e.g. [https://goerli.etherscan.io/verifyContract-solc?a=0xb7770df4fc2dd064f5867f009ce405ba3a93f5eb&c=v0.8.7%2bcommit.e28d00a7&lictype=3](https://goerli.etherscan.io/verifyContract-solc?a=0xb7770df4fc2dd064f5867f009ce405ba3a93f5eb&c=v0.8.7%2bcommit.e28d00a7&lictype=3){:target="_blank"}). The contract address should be the same one as the use for deploying the contract.

{% include elements/button.html link="https://github.com/smartinrub/ethers-smart-contract-example.git" text="Source Code" %}