# Running the Commercial Paper Smart Contract with the IBM BLockchain VSCode Extension

## Introduction

Using the new IBM Blockchain Platform IDE and the latest Hyperledger Fabric features, developing blockchain applications and smart contracts couldn't be simpler ! And what better way, than to show it in action. You can read more about it [here ](https://developer.ibm.com/announcements/ibm-blockchain-platform-vscode-smart-contract/)

The aim of this tutorial, is to enable you to deploy a sample Commercial Paper smart contract to the Fabric blockchain, using the IBM Blockchain Platform IDE and then run it. Furthermore, you'll interact with the contract, and execute transactions, using a simple command line application. The sample is available from Github https://github.com/hyperledger/fabric-samples - note there is a separate tutorial, that builds on this - adding queries to the contract, to report upon the full transaction history of a Commercial Paper stored in the ledger. Link TBA.

## Background
The Commercial Paper marketplace has been going since at least the 19th century. What is it? In short,  its a way for large institutional buyers to obtain funds, to meet short-term debt obligations. An example: MagnetoCorp issues a Commercial Paper (CP) on April 1st - face value of $1.1m. It promises to pay the bearer (the 'paper' is transferable, so can be re-sold) this amount in 6 months time (eg October 1st - the maturity date). On May 1st, the CP is purchased by an investment bank for a discounted price - $1m. If it holds this til maturity (and managing its investment risk in the meantime), then redeems it at face value ($1.1m) with MagnetoCorp, there's a profit of $100,000. This is effectively the 'interest earned' on investment of $1m for six months. Most investors tend to hold until maturity, but there are varying marketplaces, options and strategies - far beyond the scope of this little explainer !

## Scenario

Magnetocorp issue a Commercial Paper - this is performed by Isabella, an employee of MagnetoCorp. Digibank, through its investment trader Balaji, an employee, purchases the Commercial Paper. Digibank hold it for a period of time, and then redeem it at face value with MagnetoCorp for a small profit. You can read more on the Commercial paper example at https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html

The IBM Blockchain Platform IDE is available as a Visual Studio Code (VSCode) extension available from the [Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform)
. provides the developer with a u to discover, code, test, debug, package, deploy, and publish smart contracts and applications using a single tool. We created the IBM Blockchain Platform IDE as an extension to Visual Studio Code (VSCode)intuitive tools




##Learning objectives

Install the IBM Blockchain Platform VSCode extension
Create a new JavaScript smart contract
Package a smart contract
Create, explore, and understand a Hyperledger Fabric network
Deploy the smart contract on a local Hyperledger Fabric instance
Use a Node.js SDK to interact with the deployed smart contract package
## Setup a dev environment. ++

