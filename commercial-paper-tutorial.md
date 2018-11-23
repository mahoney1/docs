# Running the Commercial Paper Smart Contract with the IBM BLockchain VSCode Extension

## Introduction

Using the new IBM Blockchain Platform IDE and the latest Hyperledger Fabric features, developing blockchain applications and smart contracts couldn't be simpler ! The IDE is an intuitive tool that enables the developer to discover, code, test, debug, package, deploy, and publish smart contracts and applications in one, single tool. And what better way, than to show it in action! You can read more about it [here ](https://developer.ibm.com/announcements/ibm-blockchain-platform-vscode-smart-contract/)

The aim of this tutorial, is to enable you to deploy a sample Commercial Paper smart contract to the Fabric blockchain, using the IBM Blockchain Platform IDE and then run it. Furthermore, you'll interact with the contract, and execute transactions, using a simple command line application. The sample is available from Github https://github.com/hyperledger/fabric-samples - note there is a separate tutorial, that builds on this - adding queries to the contract, to report upon the full transaction history of a Commercial Paper stored in the ledger. Link TBA.

## Background
Commercial Paper marketplaces have been going since at least the 19th century. What is it? Well, there is a fantastic description of that in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutional buyers to obtain funds, to meet short-term debt obligations. An example: MagnetoCorp issues a Commercial Paper (CP) on April 1st - face value of $1.1m. It promises to pay the bearer (the 'paper' is transferable, so can be re-sold) this amount in 6 months time (eg October 1st - the maturity date). On May 1st, the CP is purchased by an investment bank (Digibank) for a discounted price - $1m. If it holds this til maturity (and managing its investment risk in the meantime), then, as the bearer, it can redeem it at face value ($1.1m) with MagnetoCorp - a profit of $100,000. This is effectively the 'interest earned' on an investment of $1m for six months. Most investors tend to hold until maturity, but there are varying marketplaces, options and strategies - far beyond the scope of this little explainer !

## Scenario

Magnetocorp issue a Commercial Paper - this is performed by Isabella, an employee of MagnetoCorp. An investor, Digibank - through its investment trader Balaji - purchases the Commercial Paper. Digibank hold it for a period of time, and then redeem it at face value with MagnetoCorp for a small profit. You can read more on the Commercial paper example at https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html

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

![packageFile](/pics/installExtension.gif)

You can launch VSCode from the task bar, or by typing `code` in a terminal window.

1. First thing we need to do is to install the IBM Blockchain Platform VSCode extension. You will need to install the latest version of VSCode to do this. To see if you have the latest VSCode extension, go to `Code` -> `Check for Updates`. If VSCode crashes at this point 
(which it did for me), it likely means you don't have the latest version. If so, update your VSCode, and once you're done, click on `extensions` on the side bar on the left part of your screen. At the top, search the extension marketplace for  `IBM Blockchain Platform`. Click on `Install`. You should see a status of 'Installing' and eventually 'Installed' - then click on `reload`. 

## Step 3. Open the Commercial Paper Contract

1. In VSCode, choose 'File...Open Folder' - and open the `commercial-paper` folder from in your $HOME/fabric-samples/commercial-paper directory:

2. Click on the `Explorer` icon, top left, and open the `contract` folder under `commercial-paper/organization/magnetocorp/contract`
[Smart Contract](/pics/papercontract.png)

3. Explore the file under `lib` subfolder called `papercontract.js` - this effectively orchestrates the logic for the different smart contract transaction functions (issue, buy, redeem etc) and are underpinned by some essential core functions that interact with the ledger. The link provided in the Introduction section explains the concepts, themes and programmatic approach to writing contracts, using the Commercial Paper scenario. Take some time to read that explainer.

4. Go back to the `contract` folder
[Contract folder](/pics/project-commpaper.png)


## Step 4. Package the Smart Contract 

1. Click on the `package.json` file in the Explorer palette and edit the `"name"` field - change the name to `papercontract` and save (CTRL + S) the file.
[Package Name](/pics/package-name.png)
  
2. Click on IBM Blockchain Platform sidebar icon  

3. Click on 'Add New Package' under 'Smart Contract packages' to package up the Commercial Paper smart contract package, for installing onto a peer. It will be called something like `papercontract@0.0.1`

## Step 5. Install the Smart Contract on a running Fabric

We'll start up a sample Fabric from Fabric Samples to run our smart contract. To that end, we've provided a sample `connection.json` to import into the IBM Blockchain VSCode environment - the beauty is that you can connect to the local fabric provided, or - to one you've already got up and running. 

1. You'll need to get this sample repo which has the connection.json file to import (and the Admin and User certs we'll use in our demo):

cd $HOME

`git clone https://github.com/mahoney1/commpaper`

2. Click on the 'IBM Blockchain Platform' sidebar icon - bottom left - you'll see an 'IBM Blockchain Connections' panel
3. Click the 'Add New Connection' button or icon - enter a name of 'myfabric' for the connection name then browse to find and import the `connection.json` file
4. Choose 'browse' and select the `AdminCert` for the certificate file and `Adminkey` for the key file.
5. You should now be able to click on `myfabric` and see the channel `mychannel` and click again, to see the only peer in the network.
6. Right-click on the `peer0.org1.example.com` node and elect to 'Install Smart Contract'
7. Next, highlight the channel `mychannel` and right-click and choose the `Instantiate/Upgrade Smart Contract` option - select `papercontract` as the contract to instantiate 
8. Enter `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call
9. Hit 'ENTER' - ie leave blank - when prompted to enter arguments

A cursory look of `docker ps` on the terminal window, will reveal our smart contract `papercontract` is running in its own docker container.


## Execute the Commercial Paper Smart Contract transactions from client applications - Magnetocorp and Digibank

So far, we've installed and instantiated our smart contract on the blockchain. Now its time to try out the smart contract transactions.

The Commercial paper scenario described earlier, describes running contract transactions, as employees from two different organizations, ie MagnetoCorp and Digibank.  It is already set up (in fabric-samples) the sample certificate/keys required to transact as particular blockchain identities. But the important concept here is that the relevant employee identities (certificate/ley combo) need to be imported into their private wallet. Then, a simple client application (once installed, below) will transact on the Commercial Paper `papernet` marketplace as the identity in question. Let's start with Isabella at MagnetoCorp.

### Transaction #1: Execute an issue Commercial Paper transaction as Isabella@MagnetoCorp

1. Change directory to MagnetoCorp's application directory:

`cd commercial-paper/organization/magnetocorp/application`

2. Install the NodeJS application dependencies (you may get some 'WARN's' in the output - you can ignore for now)

`npm install`

3. Run the Wallet import script - this will import the sample `User1` sample cert for this Org1 user, into an `identity/user/isabella/wallet` folder, located under the same sub-tree as the `application` folder.

`node addToWallet.js`

A simple message of 'done' is shown that the import task is completed.

4. Now execute the first Commercial paper transaction from the `application` directory - the 'issue' transaction:

`node issue.js`

You should get messages confirming it was successful:

[Issue message](/pics/issue-output.png)

### Transaction #2: Execute a 'buy' Commercial Paper transaction as Balaji@DigiBank

1. Change directory to DigiBank's application directory:

`cd commercial-paper/organization/digibank/application`

2. Install the NodeJS application dependencies (you may get some 'WARN's' in the output - you can ignore for now)

`npm install`

3. Run the Wallet import script - this will import the sample `User1` sample cert for this Org1 user, into an `identity/user/balaji/wallet` folder, located under the same sub-tree as the `application` folder.

`node addToWallet.js`

A simple message of 'done' is shown that the import task is completed.

4. Now execute the second Commercial paper transaction in its lifecycle from the `application` directory - the 'buy' transaction:

`node buy.js`

You should get messages confirming it was successful:

[Issue message](/pics/buy-output.png)

### Transaction #3: Execute a 'redeem' transaction as Balaji@DigiBank - six months later

The time has come, in this Commercial Paper's lifecycle for Digibank to redeem it - at face value and recoup their investment risk. Not surprisingly, there is a client application, called `redeem.js` which will perform this task, using Balaji's certificate, from his wallet, to perform it.

1. From the same directory `commercial-paper/organization/digibank/application` - run the redeem transaction:

`node redeem.js`

You should get messages confirming it was successful:

[Issue message](docs/pics/redeem-output.png)

Well done! You've completed the full (yet simple) lifecycle for a Commercial Paper. 

## Conclusion


Nice job - you're done. You learned how to deploy a very substantial Commercial Paper smart contract sample using the IBM Blockchain IDE and seen a glimpse of its abilities to create, package, install, instantiate a smart contract developed using Hyperledger Fabric's newest programming model. 

You've then taken that operational smart contract, and  interacted as employees of two different organizations, to complete the Commercial Paper's lifecycle. The users are using simple client applications and identities (provided by their respective organisations) and combine to write the historical evidence to the blockchain.

The next tutorial will concentrate on another application perspective: getting the history of a particular asset, like a Commercial paper, from the blockchain. This will mean adding query functionality to the contract, and consuming this as a transaction from the respective clients.

Thank you for completing this!
