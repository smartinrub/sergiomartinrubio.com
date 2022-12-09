---
title: Smart Contracts. Advanced Development
image: https://lh3.googleusercontent.com/pw/AL9nZEUI8VIcPd9rCU5oKBn8DgdwjoygUYkKcjFujQuYl7skebxH57-u4Uiq9Y7xyIYHSNnwkk8j12RQCvdisiwE2F4SbC9QKAsAHzRKIQdl12fg-MfjTV-oiD0YNzDoJ0ut-5xw9OlurrbKbOtlR2-DbtDm=w2152-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Solidity
    - Blockchain
    - Ethereum
    - Events
    - HardHat
    - Debugging
mermaid: false
layout: post
---

## Solhint

[Solhint](https://www.npmjs.com/package/solhint){:target="_blank"} is a linter for your [Solidity Smart Contracts](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/). It will flag things like unspecified visibility modifier on variables.

You can add Solhint to your project with:

```bash
yarn add --dev solhint
yarn solhint --init
```

then you can add your rules on the generated file `.solhint.json`:

```json
{
    "extends": "solhint:recommended",
    "rules": {
        "compiler-version": ["error", "^0.8.0"],
        "func-visibility": ["warn", { "ignoreConstructors": true }]
    }
}
```

To run the linter execute: `yarn solhint contracts/*.sol` and it will display something like:

```bash
   2:1   error    Compiler version ^0.8.9 does not satisfy the ^0.5.8 semver requirement                             compiler-version
  13:5   warning  Explicitly mark visibility in function (Set ignoreConstructors to true if using solidity >=0.7.0)  func-visibility
  14:9   warning  Error message for require is too long                                                              reason-string
  15:13  warning  Avoid to make time-based decisions in your business logic                                          not-rely-on-time
  27:17  warning  Avoid to make time-based decisions in your business logic                                          not-rely-on-time
  30:48  warning  Avoid to make time-based decisions in your business logic                                          not-rely-on-time

✖ 6 problems (1 error, 5 warnings)
```

## hardhat-deploy

The [hardhat-deploy plugin](https://github.com/wighawag/hardhat-deploy){:target="_blank"} makes easier the deploying process and testing of your [Smart Contracts deployed with HardHat](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/).

Some of the features are:

- Deploy contracts.
- Keep track of the deployments.
- Environment replication.
- Configuration files for deploying to multiple environments.

You can find other usages [here](https://github.com/wighawag/hardhat-deploy#what-is-it-for-){:target="_blank"}.

Two dependencies are needed:

```bash
yarn add --dev hardhat-deploy
yarn add --dev @nomiclabs/hardhat-ethers@npm:hardhat-deploy-ethers ethers
```

Then you can include them on your HardHat config file.

`hardhat.config.js`:

```js
require("@nomicfoundation/hardhat-toolbox")
require("hardhat-deploy")

module.exports = {
    solidity: "0.8.8",
}
```

Next, create a `deploy/` folder. This folder will be used by the `hardhat-deploy` plugin.

You can define a version of your smart contract given a name like `deploy/01-deploy-hello-world.js`.

```js
module.exports = async ({ getNamedAccounts, deployments }) => {
    const { deploy } = deployments
    const { deployer } = await getNamedAccounts()

    await deploy("HelloWorld", {
        from: deployer,
        args: [],
        log: true,
    })
}

module.exports.tags = ["all", "helloWorld"]
```

now you can run a node locally and all the contracts within the `deploy/` folder will get deployed.

```bash
yarn hardhat node
```

will print:

```bash
deploying "HelloWorld" (tx: 0xb92b6e2217005023fcbe3265e59b3ebea9dee0359194cf2837fa8c3383f7198d)...: deployed at 0x5FbDB2315678afecb367f032d93F642f64180aa3 with 381167 gas
```

## Solidity Style Guide

Layout contract elements in the following order:

1. Pragma statements
2. Import statements
3. Interfaces
4. Libraries
5. Contracts

Inside each contract, library or interface, use the following order:

1. Type declarations
2. State variables
3. Events
4. Modifiers
5. Functions

Functions should be grouped according to their visibility and ordered:

1. constructor
2. receive function (if exists)
3. fallback function (if exists)
4. external
5. public
6. internal
6. private

The modifier order for a function should be:

1. Visibility
2. Mutability
3. Virtual
4. Override
5. Custom modifiers

Other important conventions:

- Contracts and libraries should be named using the CapWords style
- Contract and library names should also match their filenames.
- Avoid multiple contracts on the same file.

All these rules and much more can be found on the [Official Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html){:target="_blank"}.

## Solidity Documentation

Use [NatSpec](https://docs.soliditylang.org/en/v0.8.16/natspec-format.html#natspec){:target="_blank"} guidelines for documenting your Smart Contracts.

```solidity
/**
 * @title A contract for greeting
 * @author Sergio
 * @notice This contract is a smart contract example
 * @dev This implements...
 */
 contract HelloWorld {
    // your code
 }
```

## Debugging

You can use [VSCode](https://code.visualstudio.com){:target="_blank"} for debugging. Put a break point on your JavaScript code and run the debugger from VSCode.

Another trick to debug your Solidity Smart Contracts is to use [the Solidity `console.log`](https://hardhat.org/tutorial/debugging-with-hardhat-network){:target="_blank"}.

```solidity
pragma solidity ^0.8.8;

import "hardhat/console.sol";

contract HelloWorld {
    function helloWorld() public view returns (string memory) {
        console.log("Processing hello world...");

        if (bytes(from).length == 0) {
            return "Hello World!";
        }

        return string(abi.encodePacked("Hello World from ", from, "!"));
    }
}
```

## Gas Cost and Opcodes

Depending on the operation you perform on your Smart Contract a different gas cost is associated to it. Solidity operations get translated to machine language and they are called "Opcodes". [A reference for the opcodes and their gas cost can be found on the Ethereum site](https://ethereum.org/en/developers/docs/evm/opcodes/){:target="_blank"}.

Learning about opcodes is important in order to optimize the gas consumption of your Smart Contracts.

When using storage variable the convention is to append a `s_` so we don't forget that those variables are using a lot of gas.
