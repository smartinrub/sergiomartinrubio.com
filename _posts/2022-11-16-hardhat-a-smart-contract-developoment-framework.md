---
title: Hardhat. A Smart Contract Development Framework
image: 
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Solidity
    - Blockchain
    - Ethereum
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

`contracts/MyContract.sol`

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract MyContract {
    // using pure since we are not modifying the state
    function helloWorld() public pure returns (string memory) {
        return "Hello World!";
    }
}
```

`scripts/deploy.js`

```js
const { ethers } = require("hardhat")

async function main() {
  const MyContractFactory = await ethers.getContractFactory("MyContract")
  console.log("Deploying...")
  const myContract = await MyContractFactory.deploy()
  await myContract.deployed()
  console.log(`Deployed contract to: ${myContract.address}`)
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

```js
const { ethers, run, network } = require("hardhat")

async function main() {
    const MyContractFactory = await ethers.getContractFactory("MyContract")
    console.log("Deploying contract...")
    const myContract = await MyContractFactory.deploy()
    await myContract.deployed()
    console.log(`Deployed contract to: ${myContract.address}`)
    if (network.config.chainId === 5 && process.env.ETHERSCAN_API_KEY) {
        await myContract.deployTransaction.wait(6) // make sure the contract is already on Etherscan
        await verify(myContract.address, [])
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
contracts/MyContract.sol:MyContract at 0x1CCd4f9a2838DBb19116224921E1B054C297E73f
for verification on the block explorer. Waiting for verification result...

Successfully verified contract MyContract on Etherscan.
https://goerli.etherscan.io/address/0x1CCd4f9a2838DBb19116224921E1B054C297E73f#code
```

You can check the provided link to make sure the contract is actually verified.

## Interacting with your Smart Contract

We can call the smart contract function in the same fashion as we did [on the vanilla JavaScript Smart Contract deployment script](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/#interacting-with-the-smart-contract).

```js
// imports

async function main() {
    // main function code

    const helloWorld = await myContract.helloWorld()
    console.log(helloWorld)
}

// other functions
```

>When an error is thrown you might need to delete the existing artifacts and cache and retry deploying again.
