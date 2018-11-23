# Running the Commercial Paper Smart Contract with the IBM BLockchain VSCode Extension

## Introduction

Using the new IBM Blockchain Platform IDE and the latest Hyperledger Fabric features, developing blockchain applications and smart contracts couldn't be simpler ! The IDE is an intuitive tool that enables the developer to discover, code, test, debug, package, deploy, and publish smart contracts and applications in one, single tool. And what better way, than to show it in action! You can read more about it [here ](https://developer.ibm.com/announcements/ibm-blockchain-platform-vscode-smart-contract/)

The aim of this tutorial, is to enable you to deploy a sample Commercial Paper smart contract to the Fabric blockchain, using the IBM Blockchain Platform IDE and then run it. Furthermore, you'll interact with the contract, and execute transactions, using a simple command line application. The sample is available from Github https://github.com/hyperledger/fabric-samples - note there is a separate tutorial, that builds on this - adding queries to the contract, to report upon the full transaction history of a Commercial Paper stored in the ledger. Link TBA.

## Background
The Commercial Paper marketplace has been going since at least the 19th century. What is it? In short,  its a way for large institutional buyers to obtain funds, to meet short-term debt obligations. An example: MagnetoCorp issues a Commercial Paper (CP) on April 1st - face value of $1.1m. It promises to pay the bearer (the 'paper' is transferable, so can be re-sold) this amount in 6 months time (eg October 1st - the maturity date). On May 1st, the CP is purchased by an investment bank (Digibank) for a discounted price - $1m. If it holds this til maturity (and managing its investment risk in the meantime), then, as the bearer, it can redeem it at face value ($1.1m) with MagnetoCorp - a profit of $100,000. This is effectively the 'interest earned' on investment of $1m for six months. Most investors tend to hold until maturity, but there are varying marketplaces, options and strategies - far beyond the scope of this little explainer !

## Scenario

Magnetocorp issue a Commercial Paper - this is performed by Isabella, an employee of MagnetoCorp. An investor, Digibank - through its investment trader Balaji - purchases the Commercial Paper. Digibank hold it for a period of time, and then redeem it at face value with MagnetoCorp for a small profit. You can read more on the Commercial paper example at https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html

The IBM Blockchain Platform IDE is available as a Visual Studio Code (VSCode) extension available from the [Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform)
. provides  We created the IBM Blockchain Platform IDE as an extension to Visual Studio Code (VSCode)intuitive tools

## Pre-requisites

You will need the following installed in order to use the extension:

- [Node v8.x or greater and npm v5.x or greater](https://nodejs.org/en/download/)
- [Yeoman (yo) v2.x](http://yeoman.io/)
- [Docker version v17.06.2-ce or greater](https://www.docker.com/get-started)
- [Docker Compose v1.14.0 or greater](https://docs.docker.com/compose/install/)
- [VSCode 1.28.2 or higher](https://code.visualstudio.com/download)

You can check your versions using the following commands:
- node --version
- npm --version
- yo --version
- docker --version
- docker-compose --version

- Docker for Windows is configured to use Linux containers (this is the default)

## Estimated time

After the prerequisites are installed, this should take approximately *45 minutes* to complete.

## Step 1. Get the commercial Paper Sample

1.  From a terminal window, clone the Fabric Samples repo (and specifically the 'master' branch) to your $HOME directory:

git clone -b master https://github.com/hyperledger/fabric-samples

## Step 2. Launch VSCode, install the IDE Extension

![packageFile](/docs/installExtension.gif)

You can launch VSCode from the task bar, or by typing `code` in a terminal window.

1. First thing we need to do is to install the IBM Blockchain Platform VSCode extension. You will need to install the latest version of VSCode to do this. To see if you have the latest VSCode extension, go to `Code` -> `Check for Updates`. If VSCode crashes at this point 
(which it did for me), it likely means you don't have the latest version. If so, update your VSCode, and once you're done, click on `extensions` on the side bar on the left part of your screen. At the top, search the extension marketplace for  `IBM Blockchain Platform`. Click on `Install`. You should see a status of 'Installing' and eventually 'Installed' - then click on `reload`. 

## Step 3. Open the Commercial Paper Contract

1. In VSCode, choose 'File...Open Folder' - and open the `commercial-paper` folder from in your $HOME/fabric-samples/commercial-paper directory:

2. Click on the `Explorer` icon, top left, and expand the 'contract' sub-folder to see the main contract, `papercontract.js`
[Smart Contract](docs/papercontract.png)

The `papercontract.js` effectively orchestrates the logic for the different smart contract transaction functions (issue, buy, redeem etc) and are underpinned by some essential core functions that interact with the ledger. The link provided in the Introduction section explains the concepts, themes and programmatic approach to writing contracts, using the Commercial Paper scenario. Take some time to read that explainer.

## Step 4. Package the Smart Contract 

1. Click on 'Add New Package' to package up the Commercial Paper smart contract package, for installing onto a peer.

## Step 5. Install the Smart Contract running Fabric

1. Click on the 'IBM Blockchain Platform' sidebar icon - bottom left - you'll see an 'IBM Blockchain Connections' panel
2. Click on the `local_fabric` icon to launch a local development Fabric blockchain to deploy the smart contract to
3. Right-click on `local_fabric` to toggle Development mode - this enables us to see the output of any transactions we perform
4. Click on `local_fabric` to access the channel `mychannel` - double click this to reveal the Peers and the one peer node configured
5. Right-click on the `peer0.org1.example.com` node and elect to 'Install Smart Contract'
Create a new JavaScript smart contract
Package a smart contract
Create, explore, and understand a Hyperledger Fabric network
Deploy the smart contract on a local Hyperledger Fabric instance
Use a Node.js SDK to interact with the deployed smart contract package
## Setup a dev environment. ++

