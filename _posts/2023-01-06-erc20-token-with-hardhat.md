---
title: ERC-20 Token with Hardhat
image: https://lh3.googleusercontent.com/pw/AL9nZEUE-rZE4Ni3Dg0Q-l4Q2fsMN83OUleJwUVhmDbEZmlorMhGuH5OOwhPpgVfvy8Mct9bkBurkf1yMsMTAR3KckXdHB1kJpxfH9semKBn61aT4IbMxjx4bbH35CWTPhBvzXVFipa5raORcUYn5GwtnhBk=w2460-h1640-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Ethereum
    - Smart Contract
    - Hardhat
    - OpenZeppelin
    - Solidity
    - Blockchain
    - Token
mermaid: false
layout: post
---


## What is an ERC?

An [ERC](https://eips.ethereum.org/erc){:target="_blank"} (*Ethereum Request for Comment*) is simply an application-level standard that contains a set of rules. An ERC is one type of [EIP](https://eips.ethereum.org){:target="_blank"} (*Ethereum Improvement Proposals*).

> An **EIP** describes standards for the Ethereum ecosystem, like core protocol specifications, client APIs or contract standards.

## ERC-20

One of the most popular ERCs is the [ERC-20](https://eips.ethereum.org/EIPS/eip-20){:target="_blank"} that defines token standards.

The ERC-20 specification has the following requirements:

- Solidity `0.4.17` or above.
- `false` returns must be handled.
- Defines a set of methods:
  - `function name() public view returns (string)`: the name of the token.
  - `function symbol() public view returns (string)`: the symbol of the token.
  - `function decimals() public view returns (uint8)`: the number of decimals the token uses.
  - `function totalSupply() public view returns (uint256)`: the total amount of tokens.
  - `function balanceOf(address _owner) public view returns (uint256 balance)`: the token balance of a particular account.
  - `function transfer(address _to, uint256 _value) public returns (bool success)`: transfers tokens from the caller account to a provided account.
  - `function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)`: transfers tokens from one account to another.
  - `function approve(address _spender, uint256 _value) public returns (bool success)`: allows an account to withdraw token from your account up to the provided amount.
  - `function allowance(address _owner, address _spender) public view returns (uint256 remaining)`: this is to see how much amount an account can withdraw from an owner account.
- Events must be triggered when tokens are transferred with an `Transfer` event (including 0 amount transfer) and when approvals are called successfully with a `Approval` event.

## Implementing an ERC-20 Token

You can implement an ERC-20 token by yourself in the same way as defining a Smart Contract in Solidity and making sure all the ERC-20 requirements are met.

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE
pragma solidity ^0.8.17;

contract ManualToken {
    function name() public view returns (string memory) {
        // implementation
    }

    function symbol() public view returns (string memory) {
        // implementation
    }

    function decimals() public view returns (uint8) {
        // implementation
    }

    function totalSupply() public view returns (uint256) {
        // implementation
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        // implementation
    }

    function transfer(address _from, address _to, uint256 _amount) public {
        // implementation
    }

    function transferFrom(
        address _from,
        address _to,
        uint256 _value
    ) public returns (bool success) {
        // implementation
    }

    function approve(
        address _spender,
        uint256 _value
    ) public returns (bool success) {
        // implementation
    }

    function allowance(
        address _owner,
        address _spender
    ) public view returns (uint256 remaining) {
        // implementation
    }
}
```

However, it's recommended to use one of the implementations written by companies like [OpenZeppelin](https://www.openzeppelin.com){:target="_blank"} or [ConsenSys](https://consensys.net){:target="_blank"}, so you take advantage of the gas savings and security improvements of those implementations.

>The [Consensys implementation is DEPRECATED](https://github.com/ConsenSys/Tokens) and they advice to use the OpenZeppelin implementation.

### OpenZeppelin Implementation

[OpenZeppelin provides an ERC-20 interface](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#ERC20){:target="_blank"} that you can implement for creating your own ERC-20 token. 

First of all install OpenZeppelin library:

```bash
yarn add --dev @openzeppelin/contracts
```

To use the *OpenZeppelin* implementation:
- Call the ERC20 constructor with the **token name** and **symbol**.
- Mint the initial amount of tokens. You can **initialize the initial supply of tokens** with `_mint(address account, uint256 amount)`.

`contracts/MyToken.sol`:

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor(
        string memory name,
        string memory symbol,
        uint256 initialSupply
    ) ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
    }
}

```

>By default, ERC20 uses a value of 18 for decimals.

## ERC-20 Token Testing

When extending the OpenZeppelin ERC-20 implementation to create your own token then you should write tests of any functionality that you are adding.

We are going to use HardHat and Chai for testing our token.

`test/unit/MyToken.test.ts`:

```typescript
import { expect } from "chai"
import { ethers, network } from "hardhat"
import { developmentChains } from "../../helper-hardhat-config"
import { MyToken, MyToken__factory } from "../typechain-types"

!developmentChains.includes(network.name)
    ? describe.skip
    : describe("MyToken", () => {
          const name = "OpenZeppelin Token"
          const symbol = "OZT"
          let initialSupply = "10000000000000000000000" // 10000 * 1e18

          let myTokenFactory: MyToken__factory
          let myToken: MyToken

          beforeEach(async () => {
              myTokenFactory = (await ethers.getContractFactory("MyToken")) as MyToken__factory
              myToken = await myTokenFactory.deploy(name, symbol, initialSupply)
              await myToken.deployed()
          })

          it("Should have correct name", async () => {
              expect(await myToken.name()).to.equal(name)
          })

          it("Should have correct symbol", async () => {
              expect(await myToken.symbol()).to.equal(symbol)
          })

          it("Should have correct initial supply", async () => {
              expect(await myToken.totalSupply()).to.equal(initialSupply)
          })

          it("Should have 18 decimals", async () => {
              expect(await myToken.decimals()).to.equal(18)
          })
      })
```

As you can see above we are checking that the name, symbol, initial supply and decimals are correct.

Now run the tests with `yarn hardhat test`.

>Remember to run `yarn hardhat typechain` for generating the token types (`MyToken`, `MyToken__factory`).

## ERC-20 Token Deployment

The deploying script will be the following.

`deploy/01_Deploy_MyToken.ts`:

```typescript
import { ethers } from "hardhat"
import { DeployFunction } from "hardhat-deploy/dist/types"

const deployFunction: DeployFunction = async ({ getNamedAccounts, deployments }) => {
    const { log } = deployments
    const { deployer } = await getNamedAccounts()
    const myTokenFactory = await ethers.getContractFactory("MyToken")
    let initialSupply = "10000000000000000000000" // 10000 * 1e18

    log(`Deploying token with account ${deployer}`)
    const myToken = await myTokenFactory.deploy("OpenZeppelin Token", "OZT", initialSupply)
    await myToken.deployed()
    log(`Token deployed to: ${myToken.address}`)
    log("----------------------------------------------------")
}

export default deployFunction
deployFunction.tags = [`all`, `token`]
```

And now we can deploy the token with: `yarn hardhat node`. 

If we want to deploy the token to a testnet we will run `yarn hardhat deploy --network goerli`. Then we can go to the [Goerli Etherscan page](https://goerli.etherscan.io){:target="_blank"} (e.g. [OpenZeppelin Token](https://goerli.etherscan.io/token/0x41bfa4a45d153abb6170c9863c376ec314ca5049)) and search for the token address. There you should be able to see the total supply, token name and token symbol.

{% include elements/button.html link="https://github.com/smartinrub/hardhat-erc20-example.git" text="Source Code" %}

Photo by <a href="https://unsplash.com/es/@supergios?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jonny Gios</a> on <a href="https://unsplash.com/photos/4AT3mZMuFuI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
