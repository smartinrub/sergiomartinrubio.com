---
title: How to Upgrade Smart Contracts with Hardhat
image: https://lh3.googleusercontent.com/pw/AMWts8B2tkOFXUYjJX2vMg4YduAtn9a72rZkkgg9x9Jfmt3BEVyeAap2fcm6hwRYodLo7kY2ufejlNQO0KCsAxEmII6zxeonkxtEtxoFLfPo1__dCYCE6oIDnryffncd3W356_5-Pwdt7sKE0kOPOyIUmlNc=w1836-h1376-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Solidity
    - Hardhat

mermaid: true
layout: post
---

Smart contracts are no upgradable by nature, so this means that once deployed on the blockchain, there is no way to make any modifications. There are scenarios where [deploying a new version of the smart contract](https://sergiomartinrubio.com/articles/how-to-release-new-versions-of-smart-contracts/) is necessary for multiple reasons like vulnerabilities or extension of the smart contract - and this can be done through Smart Contract proxies.

[Hardhat](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/) deploy plugin provides support for upgrading smart contracts through proxies, and we just need to specify that we are going to use a proxy on our deployment script 🙌, so let's see how it works with an example.

## Getting Started

1. Create a Hardhat project:

```bash
yarn add --dev hardhat
yarn hardhat
```

2. Import dependencies:

```bash
yarn add --dev hardhat-deploy dotenv
```

3. Update your `hardhat.config.ts` file. You can use [this one](https://github.com/smartinrub/hardhat-upgrades-example/blob/main/hardhat.config.ts) as an example (more dependencies might be missing, so you will need to import them).

4. Create your `.env` file for your environment variables.

## Creating the Contracts

We are going to create two versions of a Smart Contract. The original version of the smart contract is going to have 3 functions:

- `setValue(uint256) public`
- `getValue() public view returns (uint256)`
- `version() public pure returns (uint256)`

`contract/MyContract.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract MyContract {
    uint256 internal value;

    event ValueChanged(uint256 newValue);

    function setValue(uint256 _value) public {
        value = _value;
        emit ValueChanged(_value);
    }

    function getValue() public view returns (uint256) {
        return value;
    }

    function version() public pure returns (uint256) {
        return 1;
    }
}
```

And the second version will have the same functions as V1 plus an increment function:

- `increment() public`

`contract/MyContractV2.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract MyContractV2 {
    uint256 internal value;

    event ValueChanged(uint256 newValue);

    function setValue(uint256 _value) public {
        value = _value;
        emit ValueChanged(_value);
    }

    function getValue() public view returns (uint256) {
        return value;
    }

    function increment() public {
        value++;
    }

    function version() public pure returns (uint256) {
        return 2;
    }
}
```

## Create Deploy Script

As we already mentioned we are going to use the `hardhat-deploy` plugin for making our smart contract upgradable. You can use the [default proxy contract](https://github.com/wighawag/hardhat-deploy/blob/master/solc_0.8/proxy/EIP173Proxy.sol){:target="_blank"} by simply setting `proxy: true`, but we want to use an admin user to perform upgrades, which is recommended for [Transparent Proxies](https://sergiomartinrubio.com/articles/how-to-release-new-versions-of-smart-contracts/#transparent-proxy-pattern). Therefore, we will also need a Smart Contract Admin proxy, so we are going to use the Transparent Upgradable Proxy OpenZeppelin implementation.

Firstly, we need to add the contracts from OpenZeppelin:

```bash
yarn add --dev @openzeppelin/contracts
```

The deployment script should look like this:

`deploy/01_Deploy_MyContract.ts`

```javascript
import { DeployFunction } from "hardhat-deploy/dist/types"
import { network } from "hardhat"
import {
    developmentChains,
    VERIFICATION_BLOCK_CONFIRMATIONS,
} from "../helper-hardhat-config"
import { verify } from "../utils/verify"

const deployFunction: DeployFunction = async ({
    getNamedAccounts,
    deployments,
}) => {
    const { deploy, log } = deployments

    const { deployer } = await getNamedAccounts()
    const chainId: number | undefined = network.config.chainId
    if (!chainId) return

    const waitBlockConfirmations: number = developmentChains.includes(
        network.name
    )
        ? 1
        : VERIFICATION_BLOCK_CONFIRMATIONS
    const myContract = await deploy("MyContract", {
        from: deployer,
        args: [],
        log: true,
        waitConfirmations: waitBlockConfirmations,
        proxy: {
            proxyContract: "OpenZeppelinTransparentProxy",
        },
    })

    if (
        !developmentChains.includes(network.name) &&
        process.env.ETHERSCAN_API_KEY
    ) {
        log("Verifying...")
        await verify(myContract.address, [])
    }
    log(`----------------------------------------------------`)
}

export default deployFunction
deployFunction.tags = ["all", "my-contract", "main"]
```

When `OpenZeppelinTransparentProxy` is chosen as the `proxyContract` option, the `DefaultProxyAdmin` is also used as admin since Transparent Proxy. Alternatively, you can set the proxy admin contract with `viaAdminContract`.

```javascript
...

proxy: {
    proxyContract: "OpenZeppelinTransparentProxy",
    viaAdminContract: {
        name: "MyContractProxyAdmin",
        artifact: "MyContractProxyAdmin",
    },
},

...
```

You can also use the `OptimizedTransparentProxy` which is like `OpenZeppelinTransparentProxy` but it is optimized to not require storage read for the admin on every call.

## Deploy Upgradable Smart Contract

Now that you have your Smart Contract implementation and the deployment script you can deploy the smart contracts to a local node.

```bash
yarn hardhat node
```

The deployment command deployed three contracts:

- `DefaultProxyAdmin`: The admin contract that we will use for upgrading the implementation address.
- `MyContract_Implementation`: The deploy implementation that was renamed and appended `_Implementation`.
- `MyContract_Proxy`: The proxy contract. Calling this contract address will point to the address of `MyContract_Implementation`.

## Upgrade Script

Unfortunately, Hardhat doesn’t currently have a deployment feature for upgradable contracts, so we will have to write our own script.

The following script will:

1. Get the `DefaultProxyAdmin`, `MyContract_Proxy` and `MyContractV2`.
2. Use the `DefaultProxyAdmin` to upgrade `MyContract_Proxy` implementation address to `MyContractV2` address.

`scripts/upgrade-my-contract.ts`:

```javascript
import { ContractTransaction } from "ethers"
import { ethers } from "hardhat"
import {
    MyContract,
    MyContractV2,
    ProxyAdmin,
    TransparentUpgradeableProxy,
} from "../typechain"

async function main() {
    const proxyAdmin: ProxyAdmin = await ethers.getContract("DefaultProxyAdmin")
    const transparentProxy: TransparentUpgradeableProxy =
        await ethers.getContract("MyContract_Proxy")

    // V1
    const implementation = await proxyAdmin.getProxyImplementation(
        transparentProxy.address
    )
    const proxyMyContract: MyContract = await ethers.getContractAt(
        "MyContract", // abi
        transparentProxy.address
    )
    const contractVersion = await proxyMyContract.version()
    console.log(
        `Implementation (${implementation}) version is: ${contractVersion}`
    )

    const myContractV2: MyContractV2 = await ethers.getContract("MyContractV2")
    const upgradeTx: ContractTransaction = await proxyAdmin.upgrade(
        transparentProxy.address,
        myContractV2.address
    )
    await upgradeTx.wait(1)

    // V2
    const implementationV2 = await proxyAdmin.getProxyImplementation(
        transparentProxy.address
    )
    const proxyMyContractV2: MyContractV2 = await ethers.getContractAt(
        "MyContractV2", // V2 abi
        transparentProxy.address
    )
    const newContractVersion = await proxyMyContractV2.version()
    console.log(
        `Implementation (${implementationV2}) version is: ${newContractVersion}`
    )
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error)
        process.exit(1)
    })

```

So, let's run the upgrade script with:

```bash
yarn hardhat run scripts/upgrade-my-contract.ts --network localhost
```

## Unit Testing

Finally we can write some unit tests.

`test/unit/MyContract.spec.ts`:

```javascript
import { expect } from "chai"
import { ContractTransaction } from "ethers"
import { network, deployments, ethers } from "hardhat"
import { developmentChains } from "../../helper-hardhat-config"
import {
    MyContract,
    MyContractV2,
    ProxyAdmin,
    TransparentUpgradeableProxy,
} from "../../typechain"

!developmentChains.includes(network.name)
    ? describe.skip
    : describe("Upgrade MyContract Unit Tests", () => {
          let transparentProxy: TransparentUpgradeableProxy
          let proxyMyContractV1: MyContract
          let proxyMyContractV2: MyContractV2
          let proxyAdmin: ProxyAdmin
          beforeEach(async () => {
              await deployments.fixture(["all"])
              transparentProxy = await ethers.getContract("MyContract_Proxy")
              proxyMyContractV1 = await ethers.getContractAt(
                  "MyContract", // abi
                  transparentProxy.address
              )
              proxyMyContractV2 = await ethers.getContractAt(
                  "MyContractV2", // abi
                  transparentProxy.address
              )
              proxyAdmin = await ethers.getContract("DefaultProxyAdmin")
          })

          it("Should upgrade contract to V2", async () => {
              // GIVEN
              const contractV1Version = await proxyMyContractV1.version()
              expect(contractV1Version).to.equal(1)

              // WHEN
              const myContractV2: MyContractV2 = await ethers.getContract(
                  "MyContractV2"
              )
              const upgradeTx: ContractTransaction = await proxyAdmin.upgrade(
                  transparentProxy.address,
                  myContractV2.address
              )
              await upgradeTx.wait(1)

              // THEN
              const contractV2Version = await proxyMyContractV2.version()
              expect(contractV2Version).to.equal(2)
          })

          it("Should increment when upgraded to V2", async () => {
              // GIVEN
              const value = await proxyMyContractV2.getValue()
              expect(value).to.equal(0)
              const myContractV2: MyContractV2 = await ethers.getContract(
                  "MyContractV2"
              )
              const upgradeTx: ContractTransaction = await proxyAdmin.upgrade(
                  transparentProxy.address,
                  myContractV2.address
              )
              await upgradeTx.wait(1)

              // WHEN
              proxyMyContractV2.increment()

              // THEN
              const newValue = await proxyMyContractV2.getValue()
              expect(newValue).to.equal(1)
          })
      })
```

{% include elements/button.html link="https://github.com/smartinrub/hardhat-upgrades-example.git" text="Source Code" %}
