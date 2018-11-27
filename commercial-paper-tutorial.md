# Running the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension

## Introduction

Using the new IBM Blockchain Platform VSCode extension and the latest Hyperledger Fabric features, developing blockchain applications and smart contracts couldn't be simpler ! The extension is an intuitive tool that enables the developer to discover, code, test, debug, package, deploy, and publish smart contracts and applications in one, single tool. And what better way, than to show it in action! You can read more about it [here ](https://developer.ibm.com/announcements/ibm-blockchain-platform-vscode-smart-contract/).  Its also worth mentioning that, while we're working with a local Fabric environment below, you'll also be able to (coming soon) deploy the example you build here, to your own IBM Blockchain Platform Cloud instance.

The aim of this tutorial, is to enable you to deploy a sample Commercial Paper smart contract to the Fabric blockchain, using the IBM Blockchain Platform VSCode extension and then run it. Furthermore, you'll interact with the contract, and execute transactions, using a simple command line application. The sample is available from Github https://github.com/hyperledger/fabric-samples - note there is a separate tutorial, that builds on this - adding queries to the contract, to report upon the full transaction history of a Commercial Paper stored in the ledger. Link TBA.

## Background
Commercial Paper marketplaces have been going since at least the 19th century. What is it? Well, there is a fantastic description of that in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutional buyers to obtain funds, to meet short-term debt obligations. An example: MagnetoCorp issues a Commercial Paper (CP) on April 1st - face value of $1.1m. It promises to pay the bearer (the 'paper' is transferable, so can be re-sold) this amount in 6 months time (eg October 1st - the maturity date). On May 1st, the CP is purchased by an investment bank (Digibank) for a discounted price - $1m. If it holds this til maturity (and managing its investment risk in the meantime), then, as the bearer, it can redeem it at face value ($1.1m) with MagnetoCorp - a profit of $100,000. This is effectively the 'interest earned' on an investment of $1m for six months. Most investors tend to hold until maturity, but there are varying marketplaces, options and strategies - far beyond the scope of this little explainer !

## Pre-requisites

You will need the following installed (if not already installed) in order to use the extension:

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

## Preparation

Before starting we should do a little housekeeping. Run the following command to kill any stale or active containers:

`docker rm -f $(docker ps -aq)`

Clear any cached networks:

`docker network prune`

And lastly if you’ve already run through this tutorial or tried it previously, you’ll also want to delete the underlying chaincode image for the Commercial Paper smart contract. If you’re a user going through this content for the first time, then you won’t have this chaincode image on your system (so won't need to perform this next step). Get the container id using:

`docker ps -a |grep papercontract`

then substitute the ID and run: 
docker rmi <container_id_from above>  

It will remove any container images related to dev-peer0.org1.example.com-papercontract-0.0.1-xxxxxx

## Scenario

Magnetocorp issue a Commercial Paper - this is performed by Isabella, an employee of MagnetoCorp. An investor, Digibank - through its investment trader Balaji - purchases the Commercial Paper. Digibank hold it for a period of time, and then redeem it at face value with MagnetoCorp for a small profit. You can read more on the Commercial paper example at https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html

## Step 1. Get the commercial Paper Sample

1.  From a terminal window, clone the Fabric Samples repo (and specifically the 'master' branch) to your $HOME directory:

git clone -b master https://github.com/hyperledger/fabric-samples

## Step 2. Launch VSCode, install the IBM Blockchain Platform Extension for VSCode

![packageFile](/pics/installExtension.gif)

You can launch VSCode from the task bar, or by typing `code` in a terminal window.

1. First thing we need to do is to install the IBM Blockchain Platform VSCode extension. You will need to install the latest version of VSCode to do this. To see if you have the latest VSCode extension, go to `Code` -> `Check for Updates`. If VSCode 'exits' at this point, it likely means you don't have the latest version. If so, update your VSCode (link provided earlier), and once you're done, click on `extensions` on the side bar on the left part of your screen. At the top, search the extension marketplace for  `IBM Blockchain Platform`. Click on `Install`. You should see a status of 'Installing' and eventually 'Installed' - then click on `reload`. 

## Step 3. Open the Commercial Paper Contract

1. In VSCode, choose 'File...Open Folder' - and open the `commercial-paper` folder from in your $HOME/fabric-samples/commercial-paper directory:

2. Click on the `Explorer` icon, top left, and open the `contract` folder under `commercial-paper/organization/magnetocorp/contract`
[Smart Contract](/pics/papercontract.png)

3. Explore the file under `lib` subfolder called `papercontract.js` - this effectively orchestrates the logic for the different smart contract transaction functions (issue, buy, redeem etc) and are underpinned by some essential core functions (in the sample contract) that interact with the ledger. The link provided earlier in the Introduction section explains the concepts, themes and programmatic approach to writing contracts, using the Commercial Paper scenario. Take some time to read that explainer then resume here.

4. Go back to the `contract` folder in VSCode Explorer:
[Contract folder](/pics/project-commpaper.png)


## Step 4. Package the Smart Contract 

1. Click on the `package.json` file in the Explorer palette and edit the `"name"` field - change the name to `papercontract` and save (CTRL + S) the file.
[Package Name](/pics/package-name.png)
  
2. Click on IBM Blockchain Platform sidebar icon  

3. Click on 'Add New Package' under 'Smart Contract packages' to package up the Commercial Paper smart contract package, for installing onto a peer. It will be called something like `papercontract@0.0.1`

## Step 5. Install the Smart Contract on a running Fabric

1. We'll start up a sample Fabric runtime environment from the downloaded Hyperledger Fabric Samples to run our smart contract. To that end, we've provided a sample `connection.json` to import into the IBM Blockchain VSCode environment - the beauty is that you can connect to the local fabric provided, or - to one you've already got up and running. The Fabric sample uses its own 'basic-network' sample.

`cd $HOME/fabric-samples/basic-network`   # or wherever you've cloned the `fabric-samples` directory

`./teardown.sh  ; ./start.sh`

Wait for the output messages to indicate that the network has been started ('Successfully submitted proposal to join channel')

2. You'll also need to download the following sample Github repo, as it has the connection profile to connect to the blockchain network. You'll import it using the IBM Blockchain Platform VSCode extension (as well an Admin certificate in the same repo that we'll use in our demo):

`cd $HOME`

`git clone https://github.com/mahoney1/commpaper`

3. Click on the 'IBM Blockchain Platform' sidebar icon in VSCode - its bottom left - you'll see an 'IBM Blockchain Connections' side bar panel
4. Lets create a smart contract package from our Commercial paper project ; click on 'Add New Package' under 'Smart Contract Packages'. Confirm that a `papercontract@0.0.1` package has been created.
5. Next, collapse the 'Smart Contract Packages' using the 'twisty' and expand the 'Blockchain Connections' panel
6. Click the 'Add New Connection' button or icon - enter a name of 'myfabric' for the connection name then browse to find and import the `connection.json` file from your repo.
7. Next, 'browse' and select the `AdminCert` for the certificate file to import and 'browse... select' the `Adminkey` for the key file.
8. You should now be able to click on `myfabric` and see the channel `mychannel` become active -  click again, to see the only peer in the `basic-network` network.
9. Right-click on the peer  `peer0.org1.example.com` node and elect to 'Install Smart Contract'
10. Next, highlight the channel `mychannel` and right-click and choose the `Instantiate/Upgrade Smart Contract` option - select `papercontract` as the contract to instantiate 
11. Paste in the string `org.papernet.commercialpaper:instantiate` when prompted to 'enter a function name to call' and hit ENTER
12. Next, hit 'ENTER' - ie leave blank -when prompted to enter arguments (there are none in this case) - this will take a minute or so and you'll see a progress message in the 'output' pane.
 
The instantiation process will take about 1 minute, while it builds the chaincode container with the smart contract code and dependencies. The 'Instantiated Smart Contracts' twisty will appear inside VSCode Extension panel very shortly. Indeed, a cursory look using  `docker ps` on the terminal window, will reveal our smart contract `papercontract` is building in its own docker container.


## Step 6: Execute the Commercial Paper Smart Contract transactions from client applications - Magnetocorp and Digibank

So far, we've installed and instantiated our smart contract on the blockchain. Now its time to try out the smart contract transactions.

The Commercial paper scenario describes running contract transactions, as employees from two different organizations, ie MagnetoCorp and DigiBank.  The sample certificate/keys required to transact as two different blockchain identities are provided with the `basic-network` sample. At this point, the relevant employee identities (certificate/key combo) need to be imported into their private wallet, to use with their respective client apps, which they'll use to transact on the Commercial Paper `papernet` marketplace. Let's start with Isabella at MagnetoCorp. The diagram below summarises how she would interact as an employee of MagnetoCorp.

[Transaction flow](/pics/txflow-diagram.png)

### Transaction #1: Execute an `issue` transaction as Isabella@MagnetoCorp

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

### Transaction #2: Execute a `buy` transaction as Balaji@DigiBank

1. Change directory to DigiBank's application directory:

`cd commercial-paper/organization/digibank/application`

2. Install the NodeJS application dependencies (you may get some 'WARN's' in the output - you can ignore for now)

`npm install`

3. Run the Wallet import script - this will import the sample cert for this Org1 user, into an `identity/user/balaji/wallet` folder, located under the same sub-tree as the `application` folder.

`node addToWallet.js`

A simple message of 'done' is shown that the import task is completed.

4. Now execute the 'buy' Commercial paper transaction from the `application` directory:

`node buy.js`

You should get messages confirming it was successful:

[Issue message](/pics/buy-output.png)

### Transaction #3: Execute a second `buy` transaction as Balaji (acting on behalf of Hedgematic)

In our tutorial Hedgematic, a sister company of Digibank (in our tutorial), have bought the commercial paper from Digibank and execute a second `buy` transaction to transfer ownership yet again - we'll perform this transaction as Balaji, acting on its behalf:

1. Grab the script `buy2.js` from the github.com/mahoney1/commpaper repo that you cloned earlier and copy that file to the `commercial-paper/organization/digibank/application` 
2. From the same `commercial-paper/organization/digibank/application` directory - run the second buy transaction as follows:

`node buy2.js`


### Transaction #4: Execute a `redeem` transaction as Balaji@DigiBank - six months later

The time has come, in this Commercial Paper's lifecycle, for the Commercial paper to be redeemed by its owner (Hedgematic), at face value, and recoup the investment outlay. There is a client application, called `redeem.js` which will perform this task, and its using Balaji's certificate, from his wallet, to perform it.

1. From the same directory `commercial-paper/organization/digibank/application` - modify the `redeem.js` script - we will need to supply 'Hedgematic' on the line (approx liine 67) that begins:

`const redeemResponse = await contract.submitTransaction` 

In VSCode, change the FOURTH (4th) parameter from 'Digibank' to become 'Hedgematic' and save the file (CONTROL AND S). Commit any changes.

2. Next, run the client application to issue a `redeem` transaction

`node redeem.js`

You should get messages confirming it was successful:

[Issue message](docs/pics/redeem-output.png)

Well done! You've completed the tutorial, and successfully interacted with the Smart Contract, which demonstrates a simple lifecycle of a Commercial Paper instance (with 4 transactions) on the blockchain. 

## Conclusion


Nice job - you're done. You learned how to deploy a simple, yet substantial Commercial Paper smart contract sample using the IBM Blockchain Platform VSCode extension and seen its abilities to create, package, install, instantiate a smart contract developed using Hyperledger Fabric's newest programming model( clearly, the IBM Blockchain Platform VSCode extension provides a lot more (such as the develop/debug/test lifecycle of a developer) beyond the scope of this particular tutorial). You've interacted as employees of two different organizations, using simple client applications and wallets containing identities (provided by their respective organisations) to carry out the transactions.

The next tutorial will concentrate on another application perspective: querying the ledger, such as getting the history of transactions for a particular asset, like this Commercial paper lifecycle you've just created - in 'real life' they would use real identities, but we are using sample certificates from the crypto materials provided with the `basic-network` Fabric sample for now. This means adding query functionality to the smart contract, and, once again, running queries and getting results back to the respective application clients.

In the meantime, why not try creating your own small 'starter' smart contract using the IBP VSCode extension, by visting Horea's excellent tutorial https://developer.ibm.com/tutorials/ibm-blockchain-platform-vscode-smart-contract/

Thank you for completing this!
