---
title: Making your Smart Contract UI Decentralized
image: https://lh3.googleusercontent.com/pw/AL9nZEU1oOIZp02zyf83Xif_FcYU-NzMkIkmZYjdnjMR3Wpl_-wMQFra4B5sg9ueFCfnk42J_0BxQMs3tJTCtZBlRFgS-jZ4sJbv_X-7V1nUv-jM1hYISUjzvSaUR3A1_ECGTevH0Gy3s6TT_q0ORqzWV6cw=w2460-h1640-no?authuser=0
author: Sergio Martin Rubio
categories:
    - DevOps
    - Smart Contract
mermaid: false
layout: post
---

Traditionally Front-end is deployed on a centralized provider like *AWS*, *Google Cloud* or *Microsoft Azure*.

*But what is decentralization when we are talking about software?* it can be defined as the distribution of control to multiple nodes or entities. But don't confuse with "distributed". In a distributed network the data ownership and computer power is managed by a centralized company, whereas in a decentralized network the information ownership and arithmetic power are equally distributed among all nodes in the network.

Here we are going to show you how to deploy your [Smart Contract UI](https://sergiomartinrubio.com/articles/interact-with-smart-contracts-from-the-ui/) to a decentralized network with [IPFS](https://ipfs.tech/){:target="_blank"}.

>"The Interplanetary File System (IPFS) is a distributed file storage protocol that allows computers all over the globe to store and serve files as part of a giant peer-to-peer network." - [Cloudflare](https://developers.cloudflare.com/web3/ipfs-gateway/concepts/ipfs/){:target="_blank"}

## How does IPFS work?

The idea behind *IPFS* is that your file is hashed so you get an unique identifier that anyone can use to look up your file. *IPFS* nodes can cache a copy of your file so they become a provider of your file, similar to a [blockchain](https://sergiomartinrubio.com/articles/getting-started-with-solidity/), however *IPFS* nodes can decide what data they want to cache, and they can decide what to pin or discard to save space, so nodes only store the content they are interested in. Another difference between a blockchain and *IPFS* is that IPFS is simply a decentralized storage, therefore there is no execution.

## Hosting on IPFS

We will [install IPFS for desktop](https://docs.ipfs.tech/install/ipfs-desktop/){:target="_blank"} that comes with an IPFS node.

Once IPFS is downloaded and installed on your computer you can deploy the first file to IPFS by clicking on FILES>+IMPORT and selecting one of your files. Now you can copy the CID and paste it on your browser with the IPFS protocol (e.g. `ipfs://QmfEcMNTuDggL3hfufNN6UQwopqiS38AMwtMrLTLi9AuYx`) and you should be able to see the file. Additionally, you can get an [IPFS plugin for your browser](https://docs.ipfs.tech/install/ipfs-companion/){:target="_blank"} and have a similar setup to the desktop app.

### Making UI Static

The source code for the UI can be found below:

{% include elements/button.html link="https://github.com/smartinrub/reactjs-smart-contract-example.git" text="Source Code" %}

Add the following like to your `package.json` file:

```json
{
  // other stuff
  "homepage": "./",
  // other stuff
}

```
  
Now you can run:

  ```bash
  yarn build
  ```

A `build/` folder will be generated and open the `/build/index.html` file to make sure the static resources are working as expected.

### Deploying Static UI

Now you can import the entire `build/` folder on IPFS and run your local HardHat node `hh node`.

Hit `http://127.0.0.1:8080/ipfs/<CID>/` and you should be able to see the frontend deployed on IPFS.

### Automating the UI Deployment

We will use [fleek](https://fleek.co/){:target="_blank"} to auto deploy our UI when new commits are push to the GitHub master branch. *Fleek* uses IPFS to deploy websites and works with most modern frameworks like React, Next, Jekyll...

Steps:

- Firstly, create an account. 
- Next connect your GitHub account and select the repo with the FE code.
- Now choose IPFS as the "Hosting Services".
- The build setting should be like:
  - Framework: Create React App
  - Docker Image Name: `fleek/create-react-app:node-16`
  - Build command: `yarn && yarn build`
  - Publish directory: `build`
- Click on "Deploy site".

In case the deployment fails, like it happened to me because of the Node engine version (I had to use `yarn install --ignore-engines && yarn build`), you just need to change the settings and click on "Retry deploy".

Once the deployment is done a link to your UI is generated (e.g. [royal-sun-8817.on.fleek.co](https://royal-sun-8817.on.fleek.co){:target="_blank"}).

Other IPFS providers are:
- [NTF.STORAGE](https://nft.storage){:target="_blank"}. Fully free! 🤓
- [web3.storage](https://web3.storage){:target="_blank"}.
- [estuary.tech](https://estuary.tech){:target="_blank"}. In Alpha version at the moment (2023/01/02) and only through invitation. For people with large datasets or storing meaningful public data.

Photo by <a href="https://unsplash.com/@thomascouillard?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Thomas Couillard</a> on <a href="https://unsplash.com/photos/ug0gPPYvG1M?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  