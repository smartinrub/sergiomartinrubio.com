---
title: Interact with Smart Contracts from the UI
image: https://lh3.googleusercontent.com/pw/AL9nZEVRE6kqxinvLaChZl3svOjJ-xHVmXfh1f6bTwABDOSvwUh7x1Gb4UJyaZTrsubUMQG98CEa2vlkAheUCKfA3G-4cOYOgbIno_jXZ66lPAHgd6bBOVDb6v7xh3aDhpCh2yRUhliYFb8sw-GIuaThjX0o=w2152-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - JavaScript
    - Blockchain
    - ReactJS
    - HardHat
mermaid: false
layout: post
---

Interacting with [Smart Contracts](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/) is usually done through a user interface from your favorite browser. A crypto wallet plugin installed on your browser like [Metamask](https://metamask.io){:target="_blank"} is used a gateway between the UI and a [Solidity blockchain app](https://sergiomartinrubio.com/articles/getting-started-with-solidity/). In this article we are going to build a minium UI for interacting with a Smart Contract! 😎

## Getting Started

We are going to use the Hello World Smart contract *TypeScript* + *hardhat-deploy* branch previously created. Source code can be found below:

{% include elements/button.html link="https://github.com/smartinrub/hardhat-smart-contract-example/tree/hardhat-deploy" text="Source Code" %}

You can deploy the previous Smart Contract to a local node with:

```bash
cd hardhat-smart-contract-example
yarn hardhat node
```

We will use a separate project for the UI.

## Creating a ReactJS App

### Prerequisites

- *Node* >= 14.0.0 and *npm* >= 5.6.

Now you can create a React App with:

```bash
npx create-react-app reactjs-smart-contract-example
```

### HelloWorld React Component

We can create a `HelloWorld` React component where we will create the required UI elements and functions for interacting with a deployed Smart Contract.

`src/App.js`

```javascript
import HelloWorld from "./HelloWorld"

function App() {
    return (
        <div className="App">
            <HelloWorld />
        </div>
    )
}

export default App
```

`src/HelloWorld.js`

```javascript
const HelloWorld = () => {

    return (
        <div></div>
    )
}

export default HelloWorld

```

### Connecting with MetaMask

The first thing we can do is to check whether we have the MetaMask plugin installed on our browser or not. When MetaMask is installed on our browser a `window.ethereum` object is exposed to interact with Metamask, therefore we can simply check where this object exists or not.

As we mentioned before we are going to use [the Hello World Smart contract created on a previous article](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/) and deploy it with [the HardHat local node](https://sergiomartinrubio.com/articles/hardhat-a-smart-contract-developoment-framework/#hardhat-node). This means you will have to configure your MetaMask to get connected to the local node with one of the private keys generated when running the node. You can go to your MetaMask plugin and on the Networks dropdown click on "Add network". Next select "Add a network manually". Finally, choose a name for the network (e.g. HardHat-Localhost), paste the RPC URL from the HardHat node logs (e.g. `http://127.0.0.1:8545/`), type `31337` (default value) on the Chain ID field and `ETH` for the Currency symbol. Now click on Save and select the new network. Additionally you need to import a new account with one of the private keys from the HardHat node.

We are going to use the MUI library for styling, so go ahead and add it to your project.

```bash
yarn add --dev @mui/material @emotion/react @emotion/styled
```

```javascript
import {
    Button,
    Paper,
    Stack,
    Typography,
} from "@mui/material"
import { useState } from "react"

const HelloWorld = () => {
    const [helloWorldText, setHelloWorldText] = useState("")
    const [connectButtonText, setConnectButtonText] = useState("Connect")
    const [from, setFrom] = useState("")
    const [errorMessage, setErrorMessage] = useState(null)

    const connectToMetamask = async () => {
        if (window.ethereum) {
            try {
                await window.ethereum.request({
                    method: "eth_requestAccounts",
                })
                setConnectButtonText("Connected!")
            } catch (err) {
                console.log(err)
                setErrorMessage("There was a problem connecting to MetaMask")
            }
        } else {
            setErrorMessage("Install MetaMask")
        }
    }

    return (
        <Paper elevation={3} sx={{ p: 3 }}>
            <Stack spacing={2}>
                <Button variant="contained" onClick={connectToMetamask}>
                    {connectButtonText}
                </Button>
                {errorMessage ? (
                    <Typography variant="body1" color="red">
                        Error: {errorMessage}
                    </Typography>
                ) : null}
            </Stack>
        </Paper>
    )
}

export default HelloWorld
```

>With `await window.ethereum.request({ method: "eth_requestAccounts", })` we are requesting permission to MetaMask.

When we click on the `Connect` button MetaMask should pop up and once the connection is established the button text should change to `Connected!`.

### Interacting with a Deployed Smart Contract

We will use the `ethers` library for interacting with the Smart Contract.

```bash
yarn add --dev ethers
```

Also we need two more things:
- **Contract address**. When deploying the contract with the HardHat node it would print out something like `deploying "HelloWorld" (tx: 0xd5b77d3d545bb55e43df28deae51d1a88bde60ffaffca057c675e38384ed0ea8)...: deployed at 0x5FbDB2315678afecb367f032d93F642f64180aa3 with 381167 gas`, so in this case we need `0x5FbDB2315678afecb367f032d93F642f64180aa3`
- **ABI**. The ABI file is generated when the Smart Contract is deployed to the local node: `artifacts/contracts/HelloWorld.sol/HelloWorld.json`.

Now we have the contract address and ABI we can create a  `src/constants.js` file where we can place both.

```js
export const contractAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3"
export const abi = [
    {
        inputs: [],
        name: "helloWorld",
        outputs: [
            {
                internalType: "string",
                name: "",
                type: "string",
            },
        ],
        stateMutability: "view",
        type: "function",
    },
    {
        inputs: [
            {
                internalType: "string",
                name: "_from",
                type: "string",
            },
        ],
        name: "updateFrom",
        outputs: [],
        stateMutability: "nonpayable",
        type: "function",
    },
]
```

We can now interact with the `helloWorld` function. 
1. First we check whether MetaMask is installed or not.
2. Create instance of the provider so we can interact with the local HardHat node: `new ethers.providers.Web3Provider(window.ethereum)`.
3. Create `Contract` instance of the deployed contract given the contract address, ABI and signer: `const contract = new ethers.Contract(contractAddress, abi, signer)`.
4. Invoke the `helloWorld` Smart Contract function.

`src/HelloWorld.js`:

```javascript
import {
    Button,
    DialogTitle,
    Paper,
    Stack,
    Typography,
} from "@mui/material"
import { ethers } from "ethers"
import { useState } from "react"
import { contractAddress, abi } from "./constants"

const HelloWorld = () => {
    const [helloWorldText, setHelloWorldText] = useState("")
    const [connectButtonText, setConnectButtonText] = useState("Connect")
    const [from, setFrom] = useState("")
    const [errorMessage, setErrorMessage] = useState(null)

    // other functions...

    const helloWorld = async () => {
        if (window.ethereum) {
            try {
                const provider = new ethers.providers.Web3Provider(
                    window.ethereum
                )
                const signer = provider.getSigner()
                const contract = new ethers.Contract(
                    contractAddress,
                    abi,
                    signer
                )
                const helloWorld = await contract.helloWorld()
                setHelloWorldText(helloWorld)
            } catch (err) {
                console.log(err)
                setErrorMessage("There was a problem getting hello world")
            }
        } else {
            setErrorMessage("Install MetaMask")
        }
    }

    return (
        <Paper elevation={3} sx={{ p: 3 }}>
            <Stack spacing={2}>
                <Button variant="contained" onClick={connectToMetamask}>
                    {connectButtonText}
                </Button>
                <Button
                    variant="contained"
                    color="success"
                    onClick={helloWorld}
                >
                    Test
                </Button>
                <DialogTitle>{helloWorldText}</DialogTitle>
                {errorMessage ? (
                    <Typography variant="body1" color="red">
                        Error: {errorMessage}
                    </Typography>
                ) : null}
            </Stack>
        </Paper>
    )
}

export default HelloWorld
```

When you click on the "Test" button if everything goes well you should be able to see "Hello World!".

### Updating Smart Contract State

You can also invoke functions to change the state of a Smart Contract. In this case we are going to invoke the `updateFrom` function.

Again we are going to create an instance of the HelloWorld contract but this time we are going to call `updateFrom()` and then wait for the transaction to finish since we want to make sure the state was updated before calling `helloWorld()`.

```javascript
import {
    Button,
    DialogTitle,
    Paper,
    Stack,
    TextField,
    Typography,
} from "@mui/material"
import { ethers } from "ethers"
import { useState } from "react"
import { contractAddress, abi } from "./constants"

const HelloWorld = () => {
    const [helloWorldText, setHelloWorldText] = useState("")
    const [connectButtonText, setConnectButtonText] = useState("Connect")
    const [from, setFrom] = useState("")
    const [errorMessage, setErrorMessage] = useState(null)

    // other functions

    const updateFrom = async () => {
        if (window.ethereum) {
            try {
                const provider = new ethers.providers.Web3Provider(
                    window.ethereum
                )
                const signer = provider.getSigner()
                const contract = new ethers.Contract(
                    contractAddress,
                    abi,
                    signer
                )
                const transactionResponse = await contract.updateFrom(from)
                await listenForTransactionMine(transactionResponse, provider)
                console.log(`Done updating from to ${from}!`)
            } catch (err) {
                console.log(err)
                setErrorMessage("There was an error updating From")
            }
        } else {
            setErrorMessage("Install MetaMask")
        }
    }

    const listenForTransactionMine = async (transactionResponse, provider) => {
        console.log(`Mining ${transactionResponse.hash}...`)
        return new Promise((resolve, reject) => {
            provider.once(transactionResponse.hash, (transactionReceipt) => {
                console.log(
                    `Completed with ${transactionReceipt.confirmations} confirmations`
                )
                resolve()
            })
        })
    }

    return (
        <Paper elevation={3} sx={{ p: 3 }}>
            <Stack spacing={2}>
                <Button variant="contained" onClick={connectToMetamask}>
                    {connectButtonText}
                </Button>
                <Button
                    variant="contained"
                    color="success"
                    onClick={helloWorld}
                >
                    Test
                </Button>
                <DialogTitle>{helloWorldText}</DialogTitle>
                <TextField
                    id="outlined-basic"
                    label="From"
                    variant="outlined"
                    onChange={(e) => setFrom(e.target.value)}
                />
                <Button
                    color="secondary"
                    variant="contained"
                    onClick={updateFrom}
                >
                    Update From
                </Button>
                {errorMessage ? (
                    <Typography variant="body1" color="red">
                        Error: {errorMessage}
                    </Typography>
                ) : null}
            </Stack>
        </Paper>
    )
}

export default HelloWorld
```

## Conclusion

As you can see it's quite easy to interact with Smart Contract with JavaScript, specially because the `ethers` library previously used for the deploying scripts we can use it in the same way for building our UI.

{% include elements/button.html link="https://github.com/smartinrub/reactjs-smart-contract-example.git" text="Source Code" %}

Photo by <a href="https://unsplash.com/@halacious?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Hal Gatewood</a> on <a href="https://unsplash.com/s/photos/design?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  