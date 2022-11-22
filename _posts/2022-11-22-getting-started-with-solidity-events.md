---
title: Getting Started with Solidity Events
image: https://lh3.googleusercontent.com/pw/AL9nZEXoxoxH0TJQUkXWapLRU4A0OMi5j3gntAEq1yiCNW0WJWyNRiHURePVYPrf_bJsP-N-qulpHW2C4ecmwjSo2oAOAp4M2HBWNMgoA0PMULhu0BL6qEJp6mOImxL4j9sjKJCIq52b8fp0JBF96Hl4H-pZ=w1912-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Solidity
    - Blockchain
    - Ethereum
    - Events
    - HardHat
mermaid: false
layout: post
---

Ethereum also supports events and these can very important for notifying external applications. Events are part of the transaction receipts and they are executed by clients. Events are stored as logs in the blockchain so you can retrieve them.

>The event logs are not accessible from within [smart contracts](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/), so you need an external client to listen to these events.

[Solidity](https://sergiomartinrubio.com/articles/getting-started-with-solidity/) has a built-in type `event` that we can use for defining our events.

## Event Definition

You can define a event as part of a smart contract as follows:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract MyContract {
    event MessageSent(address indexed sender, string message);

    // functions
}
```

As you can see we used the keyword `indexed` for the sender address and this means you will be able to search for the attribute using a filter. e.g. `MessageSent({sender: <ADDRESS_TO_SEARCH>})`.

Solidity also provides another keyword for sending events named `emit`.

`contracts/MyContract.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract MyContract {
    event MessageSent(address indexed sender, string message);

    function sendMessage(string memory _message) public {
        emit MessageSent(msg.sender, _message);
    }
}
```

>Events use gas.

## Listening to Events

As we mentioned before events cannot be consumed by Smart Contract, therefore we have to build a external application.

We are going to use HardHat and run local Ethereum node to showcase how *Ethereum* events work.

Prerequisites:
- [Node.js](https://nodejs.org/en/download/){:target="_blank"}
- A project with HardHat initialized. `yarn init` and `yarn hardhat init`.

1. Run a node:

```bash
yarn hardhat node
```

2. Deploy the contract:

`scripts/deploy.ts`

```js
import { ethers } from "hardhat"

async function main() {
    const myContractFactory = await ethers.getContractFactory("MyContract")
    console.log("Deploying contract...")
    const myContract = await myContractFactory.deploy()
    await myContract.deployed()
    console.log(`Deployed contract to: ${myContract.address}`)
}

main().catch((error) => {
    console.error(error)
    process.exitCode = 1
})
```

We want to deploy the contract to the local node so update your *HardHat* config file accordingly:

`hardhat.config.ts`:

```js
import { HardhatUserConfig } from "hardhat/config"
import "@nomicfoundation/hardhat-toolbox"
import "@typechain/hardhat"
import "dotenv/config"

const config: HardhatUserConfig = {
    defaultNetwork: "hardhat",
    networks: {
        localhost: { url: "http://127.0.0.1:8545/", chainId: 31337 },
    },
    solidity: "0.8.17",
}

export default config
```

>We are also adding the `import "dotenv/config"` since we will use it for injecting some environment variables from `.env`.

Deploy the contract:

```bash
yarn hardhat run scripts/event-listener.ts --network localhost
```

>Add the contract address to your `.env`. e.g. `CONTRACT_ADDRESS=0x5FbDB2315678afecb367f032d93F642f64180aa3`

3. Start listening to events:

`scripts/listen.ts`:

```js
import { ethers } from "hardhat"

const CONTRACT_ADDRESS = process.env.CONTRACT_ADDRESS || "0xaddress"

export default async function listen() {
    const myContractFactory = await ethers.getContractFactory("MyContract")
    const myContract = await myContractFactory.attach(CONTRACT_ADDRESS)

    console.log(`Listening to events from contract: ${CONTRACT_ADDRESS}`)
    myContract.on("MessageSent", (sender, message) => {
        console.log(`Received message from ${sender} with message ${message}`)
    })
}

listen()
```

`myContract.on()` subscribes to the event `MessageSent` and logs when the event occurs.

Run the listener:

```bash
yarn hardhat run scripts/event-listener.ts --network localhost
```

4. Run the script that invokes the send message function:

`scripts/send.ts`:

```js
import { ethers } from "hardhat"

const CONTRACT_ADDRESS = process.env.CONTRACT_ADDRESS || "0xaddress"

async function main() {
    const myContractFactory = await ethers.getContractFactory("MyContract")
    const myContract = await myContractFactory.attach(CONTRACT_ADDRESS)

    console.log(`Calling perform operation on contract: ${CONTRACT_ADDRESS}`)
    await myContract.sendMessage("Hello Event!")
}

main().catch((error) => {
    console.error(error)
    process.exitCode = 1
})
```

We just simply invoke the Smart Contract function `sendMessage()` that will emit the event.

```bash
yarn hardhat run scripts/send.ts --network localhost
```

You should see on the listener process something like: "Received message from 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 with message Hello Event!" 🚀

## Testing Events

[Testing Smart Contracts](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/#testing) is a extremely important task of development process. [Mocha](https://mochajs.org){:target="_blank"} provides an API for testing events.

`test/test-deploy.ts`:

```js
import { ethers } from "hardhat"
import { expect } from "chai"
import { MyContract, MyContract__factory } from "../typechain-types"

describe("MyContract", function () {
    let myContractFactory: MyContract__factory
    let myContract: MyContract

    beforeEach(async function () {
        myContractFactory = (await ethers.getContractFactory(
            "MyContract"
        )) as MyContract__factory
        myContract = await myContractFactory.deploy()
    })

    it("Should emit event", async function () {
        let senders = await ethers.getSigners()
        await expect(myContract.sendMessage("foo"))
            .to.emit(myContract, "MessageSent")
            .withArgs(await senders[0].getAddress(), "foo")
    })
})
```

Now you can run the tests:

```bash
yarn hardhat test
```

{% include elements/button.html link="https://github.com/smartinrub/hardhat-events-example.git" text="Source Code" %}
