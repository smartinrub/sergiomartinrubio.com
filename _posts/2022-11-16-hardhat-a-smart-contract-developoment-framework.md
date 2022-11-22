---
title: Hardhat. A Smart Contract Development Framework
image: https://lh3.googleusercontent.com/pw/AL9nZEW8llJrYG9CDT35aVGVbAoXo-uYelmHenAuYijHUcw9iGlvw-hb59_4_wChNivxXLyI3TorBFSi0sJ0qEocMJWfKj6Z2S9MIJ5AO-cpRC-EW_W_RhM4lFC6a5ETT8k2yCijTnLxb3x2XtIGZwabsPwc=w2152-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Solidity
    - Blockchain
    - Ethereum
    - Testing
    - HardHat
mermaid: false
layout: post
---

A smart contract development framework makes it easy to create smart contracts fast. We have previously gone through the [Solidity fundamentals](https://sergiomartinrubio.com/articles/getting-started-with-solidity/) and [how to deploy Smart Contract with Ethers.js](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/), but now we want to speed up the process of creating a smart contracts with [Hardhat](https://hardhat.org){:target="_blank"}. We are choosing *Hardhat* because it is one of the most used smart contract development framework at the time of writing this article.

## Getting Started

1. Create a folder for a new project (e.g. `mkdir hardhat-smart-contract-example`)
2. Run `yarn init`
3. Add Hardhat package: `yarn add --dev hardhat`
4. Create Hardhat project: `yarn hardhat`. You can click on enter for all the values. It creates:
   - `contracts` folder: this is where you put your Smart Contracts.
   - `scripts` folder: this is where you put your scripts for deploying contracts or interacting with them.
   - `.gitignore`: git file for avoiding committing some file or folders.
   - `hardhat.config.js`: this is where you place the HardHat configuration, plugins, and tasks.
   - `README.md`: this is to document your project.
   - `test` folder: this is where you will create your smart contract tests.

## Deploying Your First Contract with HardHat

You can deploy a contract with HardHat by just writing a few lines compare to the JavaScript vanilla version.

`contracts/HelloWorld.sol`

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract HelloWorld {
    string from;

    function helloWorld() public view returns (string memory) {
        if (bytes(from).length == 0) {
            return "Hello World!";
        }

        return string(abi.encodePacked("Hello World from ", from, "!"));
    }

    function updateFrom(string memory _from) public {
        from = _from;
    }
}
```

`scripts/deploy.js`

```js
const { ethers } = require("hardhat")

async function main() {
    const helloWorldFactory = await ethers.getContractFactory("HelloWorld")
    console.log("Deploying...")
    const helloWorld = await helloWorldFactory.deploy()
    await helloWorld.deployed()
    console.log(`Deployed contract to: ${helloWorld.address}`)
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error)
      process.exit(1)
    })
```

as you can see you don't even have to specify the provider URL, private key or the smart contract location - HardHat is doing all of that for you 🪄

Then you can run `yarn hardhat run scripts/deploy.js`.

HardHat comes with an built-in testing environment called [HardHat Network](https://hardhat.org/hardhat-network/docs/overview){:target="_blank"} that is running in the background. You can use this dev environment for testing and debugging your scripts and smart contracts. Of course, you can change the default network and define new networks with its own values on the `hardhat.config.js` as shown below.

`hardhat.config.js`:

```js
require("@nomicfoundation/hardhat-toolbox")
require("dotenv").config()

const GOERLI_RPC_URL = process.env.GOERLI_RPC_URL
const PRIVATE_KEY = process.env.PRIVATE_KEY

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  defaultNetwork: "hardhat",
  networks: {
    // chainId from https://goerli.net
    goerli: { url: GOERLI_RPC_URL, accounts: [PRIVATE_KEY], chainId: 5 },
  },
  solidity: "0.8.8",
}
```

You can now specify the network name on the CLI `yarn hardhat run scripts/deploy.js --network goerli`.

>Remember to install `env`: `yarn add --dev dotenv`.

Now go to the [Goerli Etherscan site](https://goerli.etherscan.io){:target="_blank"} and paste the printed address, so you can see how the contract was deployed 🚀

## Programmatic Smart Contract Verification

There are three ways of verifying a smart contract, going to the Etherscan website and manually verifying the contract, [calling the Etherscan API](https://docs.etherscan.io/tutorials/verifying-contracts-programmatically){:target="_blank"} from your deploying script or using a HardHat plugin. We are going to use the [hardhat-etherscan plugin](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-etherscan){:target="_blank"}.

Next, we need an API key for Etherscan so we can use the Etherscan API. [Create an account](https://etherscan.io/register){:target="_blank"} if you don't have one. Now you can go to your account and the API keys sections and Create API Key (Add button).

Install the plugin `yarn add --dev @nomiclabs/hardhat-etherscan` and add it to your HardHat config file:

`hardhat.config.js`:

```js
// other imports
require("@nomiclabs/hardhat-etherscan")

// other env variables
const ETHERSCAN_API_KEY = process.env.ETHERSCAN_API_KEY

module.exports = {
    // other configuration
  etherscan: {
    apiKey: ETHERSCAN_API_KEY
  }
}
```

Now let's create a `verify()` function on our deploying script:

`scripts/deploy.js`:

```js
const { ethers, run, network } = require("hardhat")

async function main() {
    const helloWorldFactory = await ethers.getContractFactory("HelloWorld")
    console.log("Deploying contract...")
    const helloWorld = await helloWorldFactory.deploy()
    await helloWorld.deployed()
    console.log(`Deployed contract to: ${helloWorld.address}`)
    if (network.config.chainId === 5 && process.env.ETHERSCAN_API_KEY) {
        await helloWorld.deployTransaction.wait(6) // make sure the contract is already on Etherscan
        await verify(helloWorld.address, [])
    }
}

async function verify(contractAddress, args) {
    console.log("Verifying contract...")
    try {
        await run("verify:verify", {
            address: contractAddress,
            constructorArguments: args,
        })
    } catch (e) { // we continue in case something goes wrong
        if (e.message.toLowerCase().includes("already verified")) {
            console.log("Already verified!")
        } else {
            console.log(e)
        }
    }
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error)
        process.exit(1)
    })
```

As you can imagine you can not run the verify function against the HardHat development environment, since there is no `https://hardhat.etherscan.io`. This means we have to filter out the HardHat network. We can do it by checking the `chainId` as you can see above.

Again, run `yarn hardhat run scripts/deploy.js --network goerli` and after a while you should see on the console something like:

```
Successfully submitted source code for contract
contracts/HelloWorld.sol:HelloWorld at 0x1CCd4f9a2838DBb19116224921E1B054C297E73f
for verification on the block explorer. Waiting for verification result...

Successfully verified contract HelloWorld on Etherscan.
https://goerli.etherscan.io/address/0x1CCd4f9a2838DBb19116224921E1B054C297E73f#code
```

You can check the provided link to make sure the contract is actually verified.

## Interacting with your Smart Contract

We can call the smart contract function in the same fashion as we did [on the vanilla JavaScript Smart Contract deployment script](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/#interacting-with-the-smart-contract).

`scripts/deploy.js`

```js
// imports

async function main() {
    // main function code

    const result = await helloWorld.helloWorld()
    console.log(result)

    // update hello world
    const transactionResponse = await helloWorld.updateFrom("Sergio")
    await transactionResponse.wait(1)
    const updatedResult = await helloWorld.helloWorld()
    console.log(updatedResult)
}

// other functions
```

>When an error is thrown you might need to delete the existing artifacts and cache (`yarn hardhat clean`) and retry deploying again.

## HardHat Tasks

You can define HardHat tasks in addition to the existing ones that are shown when running `yarn hardhat`. Tasks are usually created as plugins, whereas scrips are for your development.

Tasks are defined on the `hardhat.config.js` file.

```js
require("@nomicfoundation/hardhat-toolbox");

task("my-task", "This is a custom task").setAction(async () => {console.log("Hello Task!")});

// other stuff
```

now run `yarn hardhat` and you will see `my-task   This is a custom task` was added to the list of tasks. Now run it `yarn hardhat my-task` and `Hello Task!` will be printed out.

>The convention is to create tasks on their own folder: `task/my-task.js`, then import those tasks on the `hardhat.config.js` file (`require("./tasks/my-task")`).

You can also do more advance stuff like printing out the `chainId`.

`tasks/chain-id.js`:

```js
const { task } = require("hardhat/config")

task("chain-id", "Prints Chain ID").setAction(async (taskArgs, hre) => {
    console.log(hre.network.config.chainId)
})

module.exports = {}
```

`hre` provides a wide range of operations that allows you to access network configuration, interact with Ethers.js and other cool stuff.

## HardHat Node

You can spin up your own node, similar to the [Ganache node](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/#spin-up-a-local-ethereum-blockchain), by running `yarn hardhat node`. At the time of writing these lines the node comes with 20 accounts.

You can give it a try by running the node and configuring an additional network.

`hardhat.config.js`:

```js
module.exports = {
  defaultNetwork: "hardhat",
  networks: {
    // other networks
    localhost: {url: "http://127.0.0.1:8545/", chainId: 31337}
  },
  solidity: "0.8.8",
  // other stuff
}
```

now when you run your deploying script `yarn hardhat run scripts/deploy.js --network localhost` the contract will be created on the local node.

```
  Contract deployment: HelloWorld
  Contract address:    0x5fbdb2315678afecb367f032d93f642f64180aa3
  Transaction:         0xa0a3c483d79a4dc77d602a8d7a4bb18768990570cebfe9f43856adbd077392e4
  From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
  Value:               0 ETH
  Gas used:            135067 of 135067
  Block #1:            0xeb8fa4f7d50f4c1069756a939e8624823c89114a5854d69f88f87bdb90ab4663
  
  ...

  Contract call:       HelloWorld#helloWorld
  From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
  To:                  0x5fbdb2315678afecb367f032d93f642f64180aa3
```

## Interacting with Networks via CLI

Run: `yarn hardhat console --network localhost` (assuming the local node network name is `localhost`) or `yarn hardhat console --network goerli`.

You can invoke the functions as the ones you run on your scripts.

## Testing

HardHat works with [Mocha](https://mochajs.org){:target="_blank"}, which is a *JavaScript* framework.

Test are placed on the `/test` folder.

`test/test-deploy.js`

```js
const { ethers } = require("hardhat")
const { expect, assert } = require("chai")

describe("HelloWorld", function () {
    let helloWorldFactory, helloWorld

    beforeEach(async function () {
        helloWorldFactory = await ethers.getContractFactory("HelloWorld")
        helloWorld = await helloWorldFactory.deploy()
    })

    it("Should print Hello World", async function () {
        const result = await helloWorld.helloWorld()
        const expected = "Hello World!"
        // assert.equal(result, expected)
        expect(result).to.equal(expected)
    })

    it("Should update Hello World with from", async function () {
        await helloWorld.updateFrom("Sergio")
        const result = await helloWorld.helloWorld()
        const expected = "Hello World from Sergio!"
        expect(result).to.equal(expected)
    })
})
```

You can run it with `yarn hardhat test`.

Run a particular test with `yarn hardhat test --grep Hello`, which would run all the tests that contain `Hello` on their name. Alternatively, you can add the `only()` keyword to run a particular test (`it.only("Should print Hello World", async function () {}`).

## Additional HardHat Features

### Gas Reporter

Add the HardHat Gas Reporter plugin to see how much gas your are using.

```bash
yarn add --dev hardhat-gas-reporter
```

enable it:

`hardhat.config.js`:

```js
// other imports
require("hardhat-gas-reporter")

// other constants
const COINMARKETCAP_API_KEY = process.env.COINMARKETCAP_API_KEY

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {

    // other stuff

    gasReporter: {
        enabled: process.env.REPORT_GAS ? true : false,
        outputFile: "gas-report.txt",
        noColors: true,
        currency: "EUR",
        coinmarketcap: COINMARKETCAP_API_KEY,
        token: "BNB", // deploying to Binance. Default is ETH
    },
}
```

As you can see in the previous code snippet we are using [CoinMarketCap](https://coinmarketcap.com){:target="_blank"} so we can get the gas price in `EUR`. You can get an API key signing up [here](https://pro.coinmarketcap.com){:target="_blank"}.

Now run your tests: `REPORT_GAS=true yarn hardhat test`. It will save into a `gas-report.txt` file something like this:

```text
·-----------------------------|----------------------------|-------------|-----------------------------·
|     Solc version: 0.8.8     ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │
······························|····························|·············|······························
|  Methods                    ·               13 gwei/gas                ·       254.64 eur/bnb        │
···············|··············|··············|·············|·············|···············|··············
|  Contract    ·  Method      ·  Min         ·  Max        ·  Avg        ·  # calls      ·  eur (avg)  │
···············|··············|··············|·············|·············|···············|··············
|  HelloWorld  ·  updateFrom  ·           -  ·          -  ·      44997  ·            1  ·       0.15  │
···············|··············|··············|·············|·············|···············|··············
|  Deployments                ·                                          ·  % of limit   ·             │
······························|··············|·············|·············|···············|··············
|  HelloWorld                 ·           -  ·          -  ·     381167  ·        1.3 %  ·       1.26  │
·-----------------------------|--------------|-------------|-------------|---------------|-------------·
```

so we would spend €1.26 if we deploy our contract to the Binance network. The call to `updateFrom` has a cost of €0.15.

### Code Coverage

Add the Solidity HardHat code coverage plugin:

```
yarn add --dev solidity-coverage
```

then run:

```
yarn hardhat coverage
```

and it will generate a `coverage.json` file and print something like this:

```
-----------------|----------|----------|----------|----------|----------------|
File             |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-----------------|----------|----------|----------|----------|----------------|
 contracts/      |      100 |      100 |      100 |      100 |                |
  HelloWorld.sol |      100 |      100 |      100 |      100 |                |
-----------------|----------|----------|----------|----------|----------------|
All files        |      100 |      100 |      100 |      100 |                |
-----------------|----------|----------|----------|----------|----------------|
```

This is important so you make sure your code is fully tested.

{% include elements/button.html link="https://github.com/smartinrub/hardhat-smart-contract-example.git" text="Source Code" %}
