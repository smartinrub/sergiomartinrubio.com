---
title: How to Create and Mint an NFT with Hardhat
image: https://lh3.googleusercontent.com/pw/AL9nZEXGw13y5SoSAgKS2_wLRC1eMlRrbeXRnPKrafpbGZLQowA2W2ZBrUIFPP-avwYyAj22Dh5CCqNQLe1i3JYYuq0eYaxqi0exSzHHzu2HgnyJ3Nx-TrqQNVxGccXLOfPuok9XwXnqqtwtVlvtuESau6qR=w1280-h850-no?authuser=0
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

A NFT (Non-Fungible Token) is a unique token that cannot be replicated unlike fungible tokens like [ERC-20](https://sergiomartinrubio.com/articles/mint-your-first-erc20-token-with-hardhat-and-openzeppelin/), this means NFTs cannot be used for commercial transactions. Usually NFTs represent some kind of digital art or complement  real-world items, like digital tickets, the ownership of intellectual property, patterns or sensitive data (e.g. medical records, real estate ownership, music), ensuring the authenticity of a product, tracking (e.g. supply chain) or gaming (e.g. trading of weapons).

## ERC-721

A Non-Fungible Token is just another Ethereum standard, the [EIP-721](https://eips.ethereum.org/EIPS/eip-721){:target="_blank"}.

A NFT contract must implement the following interfaces:
- `ERC721`. Defines the following events and methods:
  - `event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId`
  - `event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId`
  - `event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved`
  - `function balanceOf(address _owner) external view returns (uint256)`
  - `function ownerOf(uint256 _tokenId) external view returns (address)`
  - `function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable`
  - `function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable`
  - `function transferFrom(address _from, address _to, uint256 _tokenId) external payable`
  - `function approve(address _approved, uint256 _tokenId) external payable`
  - `function setApprovalForAll(address _operator, bool _approved) external`
  - `function getApproved(uint256 _tokenId) external view returns (address)`
  - `function isApprovedForAll(address _owner, address _operator) external view returns (bool)`
- [ERC165](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-165.md){:target="_blank"}: This is just to identify what interface our smart contract is implementing, so we check whether the provided `interfaceID` matches the configured interface ID or not, and return `true` or `false` accordingly. In our case `supportsInterface()` should return `true` when the provided `interfaceID` is `0x80ac58cd`.
  - `function supportsInterface(bytes4 interfaceID) external view returns (bool)`
- `ERC721TokenReceiver` (mandatory only when using `safeTransferFrom`). This is an enquiry done by the smart contract caller to know if the ERC-721 implementation is aware of the necessity of implementing `ERC721` functionalities, so the token is safe and can be transferred out (it doesn't become forever locked in the contract! 😱), however, this doesn't mean the previous assurances are true, and only an awareness of the necessity. The receiving contract is therefore responsible to take proper actions upon receiving NFTs.
  - `function onERC721Received(address _operator, address _from, uint256 _tokenId, bytes _data) external returns(bytes4)`

Optionally you can also implement the `ERC721Metadata` interface to fetch information about the NFT:
-  `function name() external view returns (string _name)`: returns the name of the collection of NFT tokens.
-  `function symbol() external view returns (string _symbol)`: returns the abbreviated name of the collection of NFTs.
-  `function tokenURI(uint256 _tokenId) external view returns (string)`: returns the asset linked to a particular NFT within the collection of NFTs.

## Implementing a Non-Fungible Token

We are going to use the [OpenZeppelin ERC721 interface](https://docs.openzeppelin.com/contracts/4.x/){:target="_blank"}.

Firstly, install *OpenZeppelin* contracts library:

```bash
yarn add --dev @openzeppelin/contracts
```

The OpenZeppelin implementation provides a set of interfaces and implementation for creating NFTs. We are going to use `_safeMint(address to, uint256 tokenId)` to create a new NFT and accepts an address and a token ID.

`contracts/MyNFT.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    uint256 private s_tokenCounter; // if you have a collection of tokens on the same smart contract each of them needs their own unique token ID

    constructor() ERC721("MyNFT", "MNFT") {
        s_tokenCounter = 0;
    }

    function mintNFT() public returns (uint256) {
        _safeMint(msg.sender, s_tokenCounter); // this
        s_tokenCounter++; // each time we mint an NFT we increase the token counter
        return s_tokenCounter;
    }

    function getTokenCounter() public view returns (uint256) {
        return s_tokenCounter;
    }
}
```

As you can see above every time a new token is created we need to increase the token counter, and this is different to what we had to do for the [ERC20](https://sergiomartinrubio.com/articles/mint-your-first-erc20-token-with-hardhat-and-openzeppelin/#what-is-an-erc) tokens, where we initialize an initial supply of tokens when minting the ERC-20. For the `ERC721` we have to mint tokens one by one.

The *OpenZeppelin* implementation of the `ERC721` is storing the token ID and owner address when calling `_safeMint()`, and finally it emits a `Transfer` event.

The previous definition of the NFT is not linked to a particular asset, so that's what we are going to do next.

## Linking an Asset to an NFT

### Uploading NFT Asset

First of all we are going to upload an asset to a NFT storage provider. We are going to use [NFT.STORAGE](https://nft.storage){:target="_blank"}.

There are 3 options for uploading assets to *NFT.STORAGE*:

1. Via the web app. Create an account and upload an asset by clicking on "+Upload". Now you can click on "Actions" and copy [the IPFS (Interplanetary File System)](https://sergiomartinrubio.com/articles/making-your-smart-contract-ui-decentralized/#how-does-ipfs-work) URL or the HTTPS URL.
2. Via the [NFT.STORAGE app.](https://nft.storage/docs/how-to/nftup/){:target="_blank"}.
3. Programmatically! 🤓.

Since this is a coding tutorial let's go for the programmatic approach.

1. Get an API token. Once logged in, go to API Keys and generate a new one.
2. Create an environment variable `NFT_STORAGE_KEY=<key>` in your `.env` file and paste the API key.
3. Install the NPM NFT.STORAGE library.

```bash
yarn add --dev nft.storage mime
```

3. Create a folder for the image: `mkdir images` and put the image inside the folder.
4. Create the upload script. [NFT.STORAGE docs](https://nft.storage/docs/#using-the-javascript-api){:target="_blank"}.

`utils/upload.mjs`:

```javascript
const { NFTStorage, File } = require("nft.storage")
const mime = require("mime")
const fs = require("fs")
const path = require("path")

require("dotenv").config()

const NFT_STORAGE_KEY = process.env.NFT_STORAGE_KEY || "key"

async function storeNFT(imagePath, name, description) {
    const image = await fileFromPath(imagePath)

    const nftstorage = new NFTStorage({ token: NFT_STORAGE_KEY })

    const response = await nftstorage.store({
        image,
        name,
        description,
    })
    console.log(`NFT image ${response.data.name} uploaded: ${response.url}`)
    return response
}

async function fileFromPath(filePath) {
    const content = await fs.promises.readFile(filePath)
    const type = mime.getType(filePath)
    return new File([content], path.basename(filePath), { type })
}

module.exports = { storeNFT }
```

We will run the previous script as part of the NFT contract deployment script.

### Adding the Token URL to Your NFT Contract

Now we can update our NFT smart contract to inject the NFT image URL.

`contracts/MyNFT.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

error MyNFT__AlreadyInitialized();

contract MyNFT is ERC721 {
    uint256 private s_tokenCounter; // if you have a collection of tokens on the same smart contract each of them needs their own unique token ID
    string internal s_tokeUri;
    bool private s_initialized;

    constructor(string memory tokenUri) ERC721("MyNFT", "MNFT") {
        s_tokenCounter = 0;
        _initializeContract(tokenUri);
    }

    function mintNFT() public returns (uint256) {
        _safeMint(msg.sender, s_tokenCounter); // this
        s_tokenCounter++; // each time we mint an NFT we increase the token counter
        return s_tokenCounter;
    }

    function tokenURI(
        uint256 /*tokenId*/
    ) public view override returns (string memory) {
        return s_tokeUri; // using the same image for all minted tokens
    }

    function _initializeContract(string memory tokenUri) private {
        if (s_initialized) {
            revert MyNFT__AlreadyInitialized();
        }
        s_tokeUri = tokenUri;
        s_initialized = true;
    }

    function getTokenCounter() public view returns (uint256) {
        return s_tokenCounter;
    }
}
```

## Deployment

Now we can write our deployment script to deploy the NFT to your local network, a testnet or mainnet.

`deploy/01-deploy-my-nft.js`:

```javascript
const { network } = require("hardhat")
const {
    developmentChains,
    VERIFICATION_BLOCK_CONFIRMATIONS,
} = require("../helper-hardhat.config")
const { verify } = require("../utils/verify")
const { storeNFT } = require("../utils/upload")

let tokenUri =
    "ipfs://bafkreia6pu2v77h6qdvgssaw5uljatm5hjdl3ec5d24k3isb2bna3nyfkq"

module.exports = async ({ getNamedAccounts, deployments }) => {
    const { deploy, log } = deployments

    const { deployer } = await getNamedAccounts()

    const waitBlockConfirmations = developmentChains.includes(network.name)
        ? 1
        : VERIFICATION_BLOCK_CONFIRMATIONS
    log(`----------------------------------------------------`)

    if (process.env.UPLOAD_TO_NFT_STORAGE == "true") {
        tokenUri = await uploadTokenImage()
    }

    const arguments = [tokenUri]
    const myNFT = await deploy("MyNFT", {
        from: deployer,
        args: arguments,
        log: true,
        waitConfirmations: waitBlockConfirmations,
    })

    // Verify the deployment
    if (
        !developmentChains.includes(network.name) &&
        process.env.ETHERSCAN_API_KEY
    ) {
        log("Verifying...")
        await verify(myNFT.address, arguments)
    }
    log(`----------------------------------------------------`)
}

async function uploadTokenImage() {
    const response = await storeNFT(
        "./images/smr-logo.png",
        "My NFT",
        "This is an image for my NFT"
    )
    return response.url
}

module.exports.tags = ["all", "mynft", "main"]
```

To deploy to the *Goerli* testnet and upload the image to *NFT.STORAGE*:

```bash
UPLOAD_TO_NFT_STORAGE=true hh deploy --network goerli
```

The output should be something like:

```bash
----------------------------------------------------
NFT image My NFT uploaded: ipfs://bafyreies6zqyooxmvc7rnpnvhglc2dvkcflboedp6exqhbiavkfmlv5vz4/metadata.json
deploying "MyNFT" (tx: 0x4e33d8634bb9b91bc99834c94f7ca9ed20c70573480c47a4bdcd8d79d8e1f35a)...: deployed at 0xbe529C226d04bF1B44E44BDe175d422902cE8002 with 2248183 gas
Verifying...
Verifying contract...
Nothing to compile
Successfully submitted source code for contract
contracts/MyNFT.sol:MyNFT at 0xbe529C226d04bF1B44E44BDe175d422902cE8002
for verification on the block explorer. Waiting for verification result...

Successfully verified contract MyNFT on Etherscan.
https://goerli.etherscan.io/address/0xbe529C226d04bF1B44E44BDe175d422902cE8002#code
```

If you go to the Goerli Etherscan link you can the token URI which contains all the metadata associated to the NFT. See the example below:

`ipfs://bafyreies6zqyooxmvc7rnpnvhglc2dvkcflboedp6exqhbiavkfmlv5vz4/metadata.json`:

```json
{
  "name": "My NFT",
  "description": "This is an image for my NFT",
  "image": "ipfs://bafybeigxde2t2koxbvj3xojtrmrwk2gxguivpvic7ujot55ptk4z6iefxy/smr-logo.png"
}
```

## Testing

Finally we can create some unit tests to make sure the NFT is working as expected.

`test/myNFT.test.js`:

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

error MyNFT__AlreadyInitialized();

contract MyNFT is ERC721 {
    uint256 private s_tokenCounter; // if you have a collection of tokens on the same smart contract each of them needs their own unique token ID
    string internal s_tokeUri;
    bool private s_initialized;

    constructor(string memory tokenUri) ERC721("MyNFT", "MNFT") {
        s_tokenCounter = 0;
        _initializeContract(tokenUri);
    }

    function mintNFT() public returns (uint256) {
        _safeMint(msg.sender, s_tokenCounter); // this
        s_tokenCounter++; // each time we mint an NFT we increase the token counter
        return s_tokenCounter;
    }

    function tokenURI(
        uint256 /*tokenId*/
    ) public view override returns (string memory) {
        return s_tokeUri; // using the same image for all minted tokens
    }

    function _initializeContract(string memory tokenUri) private {
        if (s_initialized) {
            revert MyNFT__AlreadyInitialized();
        }
        s_tokeUri = tokenUri;
        s_initialized = true;
    }

    function getTokenCounter() public view returns (uint256) {
        return s_tokenCounter;
    }
}
```

Congratulations you were able to create an NFT with a decentralized image 🚀.

{% include elements/button.html link="https://github.com/smartinrub/hardhat-nft-example.git" text="Source Code" %}
