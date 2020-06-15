---
# Front matter (metadata).

abstract: "Deploy a sample commercial paper smart contract to a v2 Hyperledger Fabric blockchain using the IBM Blockchain Platform VS Code extension, and then run it."

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

completed_date: "2020-06-21"

components:
  - "hyperledger-fabric"
  - "hyperledger"
  - "vscode-extension"

draft: true 

excerpt: "Learn how to deploy and run a sample commercial paper smart contract using Hyperledger Fabric and the IBM Blockchain Platform VS Code extension."

meta_description: "Run a sample commercial paper smart contract on a Hyperledger Fabric v2 blockchain using the IBM Blockchain Platform VS Code extension."

meta_keywords: "commercial paper, smart contract, IBM Blockchain, VS Code extension, Hyperledger Fabric"

last_updated: "2020-06-21"

primary_tag: "blockchain"

pta:
 - "emerging technology and industry"

pwg:
  - "blockchain"

related_content:
  - type: announcements
    slug: ibm-blockchain-platform-vscode-smart-contract
  - type: articles
    slug: make-smart-contracts-smarter-with-analytics
  - type: patterns
    slug: create-and-execute-blockchain-smart-contracts

related_links:
  - title: "Video: Start developing with the IBM Blockchain Platform VS Code Extension"
    url: "https://youtu.be/0NkGGIUPhqk"
  - title: "Sample commercial paper smart contract"
    url: "https://github.com/hyperledger/fabric-samples"
  - title: "Hyperledger Fabric docs: Commercial paper tutorial"
    url: "https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html"

# runtimes:

# series:                 # OPTIONAL
#  - type:
#    slug:

services:
 - "blockchain"

subtitle: "Develop blockchain apps and smart contracts easily with the IBM Blockchain Platform extension and the latest Hyperledger Fabric features"

tags:
  - "finance"

title: "Run a commercial paper smart contract with the IBM Blockchain VS Code extension"

type: tutorial

private_portals:
  - "blockchain"

---

Using the new [IBM Blockchain Platform VS Code extension](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) and the latest Hyperledger Fabric features, developing blockchain applications and smart contracts couldn't be simpler! The extension is a powerful and intuitive tool that enables developers to develop use cases, and start developing your pilot projects. The extension allows the application developer to easily generate, code, test, debug, package, and deploy smart contracts to a blockchain network and create client applications - in a single tool. What better way to illustrate than to show it in action! 

This tutorial shows you how to deploy a sample Commercial Paper smart contract to a Fabric-based blockchain network and try out its transactions, using the IBM Blockchain Platform VS Code extension. You'll also interact with the contract and execute transactions using Node.js client applications. The sample is available from Fabric Samples on [GitHub](https://github.com/hyperledger/fabric-samples).

**Note:** This is the first of a three-part [tutorial series](https://developer.ibm.com/series/blockchain-running-enhancing-commercial-paper-smart-contract/). [Part 2](https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension/) builds on this, adding queries to the contract to report on the full transaction history of a commercial paper stored in the ledger.

**Figure 1: “Papernet” — overview of Commercial Paper member organizations used in this tutorial series**
![Overview of commercial paper member organizations used in this tutorial series](images/series-diagram.png)

## Background

Commercial paper has existed since at least the 19th century. What is it? Well, for a detailed description, check out the [Fabric Developing Applications](https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) pages. In short, it's a way for large institutions to obtain funds to meet short-term debt obligations. For example,  let's say MagnetoCorp issues a commercial paper (or "CP") on April 1 with a face value of $1m. It promises to pay the bearer this amount in 6 months' time (October 1, the maturity date) - the "paper" is a transferable asset, so it can be re-sold in a marketplace. On May 1,  the CP is purchased by an investment bank (DigiBank) for a discounted price -- $0.96 million; MagnetoCorp gets the liquidity it needs. If DigiBank holds this until maturity (managing its investment risk in the meantime), it can redeem it at face value ($1 million) with MagnetoCorp -- a profit of $40,000. This is equivalent to guaranteed interest earned on an investment of $0.96m over 6 months. Most investors in the commercial paper marketplace tend to hold it until maturity, but there are varying marketplaces, options, and strategies ... far beyond the scope of this little explainer.

## Pre-requisites

The list of installation pre-requisites for the [IBM Blockchain Platform VS Code extension](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform#user-content-dependency-installation)

Also check out the extension in the [marketplace](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) for more information.

You can check your versions using the following commands:

* `node --version`
* `npm --version`
* `code --version`

## Estimated time

After the prerequisites are installed, this should take approximately 45 minutes to complete.

## Preparation

Before starting, you need to do a little housekeeping. Run the following command to kill any stale or active containers in your development environment:

```
docker rm -f $(docker ps -aq)
```

Clear any cached networks and volumes:

```
docker network prune ; docker volume prune
```

## Scenario

MagnetoCorp manufactures electric vehicles and has just landed a big contract. They have a short turnaround time, so will sub-contract most of the work; this means they need funds/liquidity to be able to pay contractors weekly. MagnetoCorp have been here before. They will issue a commercial paper for sale at $1m to obtain funds -  this is performed by Isabella, a MagnetoCorp employee. A few weeks later, an investor, DigiBank (through its investment trader, Balaji) has an offer of $0.96m accepted on the advertised commercial paper. DigiBank holds it for a period of time (eg 6 months), and then redeems it at face value with MagnetoCorp, gaining a return on investment. Note that a commercial paper can 'change hands' a number of times in a real marketplace. You can read more about this commercial paper example in [this Hyperledger Fabric docs tutorial](https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html).

## Steps

### Step 1. Get the commercial paper sample

From a terminal window, clone the Fabric samples repo (and specifically the "master" branch) to your $HOME directory:

```
git clone https://github.com/hyperledger/fabric-samples
```

### Step 2. Launch VS Code and install the IBM Blockchain Platform extension for VS Code

You can launch VS Code from your task bar, or by typing `code` in a terminal window.

Now you need to install the IBM Blockchain Platform VS Code extension -- you'll need to install the minimum version VS Code (see pre-requisites) to do this successfully. To see if you have the right version, go to `Help` -> `Check for updates`. Next, click on the `Extensions` icon in the VS Code sidebar (left). Then, in the search bar, type `IBM Blockchain Platform` and click on `Install`. You should see a status of "Installing" and eventually "Installed" -- click `reload` when prompted.

**Figure 2. Find and install the extension from VS Code marketplace**

![Find and install the extension](images/installExtension.gif)

### Step 3. Open the commercial paper contract

1. In VS Code, choose **File** > **Open Folder**, and select the `contracts` folder by navigating to the `$HOME/fabric-samples/commercial-paper/organization/magnetocorp` directory. This is your top-level project folder for this tutorial.

2. Click on the `Explorer` icon (top left) and open the `contract` folder under `$HOME/fabric-samples/commercial-paper/organization/magnetocorp/`.
  
   **Figure 3. Open the commercial paper sample project in VS Code**
   ![Open the commercial paper sample project in VS Code](images/papercontract.png)

3. Explore the `papercontract.js` file, which is located in the `lib` subfolder. It effectively orchestrates the logic for the different smart contract transaction functions (issue, buy, redeem, etc.), and is underpinned by essential core functions (in the sample contract) that interact with the ledger. The link provided in the introduction section above explains the concepts, themes, and programmatic approach to writing contracts using the commercial paper scenario. Take some time to read that explainer and then resume here.

4. Go back to the `contract` folder by clicking on the folder name on the left in the VS Code Explorer. It's important to do so before the next step.
  
   **Figure 4. Choose the contract folder**
   ![Choose the contract folder](images/project-commpaper.png)

### Step 4. Package the smart contract

1. Click inside `package.json` in the Explorer palette and edit the "name" field; change the name to `papercontract`.
  
   **Figure 5. Editing the package.json file**
  
   ![Editing the package.json file](images/package-name.png)

2. Edit these existing "dependencies" entries (as necessary). They should read as follows:
  
   ```
     "fabric-contract-api": "~1.4.0",
    "fabric-shim": "~1.4.0"
   ```
  
   Now save (CTRL + S) the file.

3. Click on the IBM Blockchain Platform sidebar icon. When you do this the first time, you may get a message that the extension is "activating" in the output pane.

4. Click on the "Smart Contracts" sub-menu to expand. Then click on the ellipsis (“...”) button and choose "Package Open Project" for installing onto a peer. The package will be called something like `papercontract@0.0.1`.

### Step 5. Install the smart contract on a running Fabric

1. Using the IBM Blockchain Platform from the left sidebar, start up a local development runtime Fabric environment -- the IBM Blockchain Platform VS Code extension conveniently provides these local dev operations (such as start, stop, teardown, restart, etc.) for you. If you've already got an IBM Blockchain Platform VS Code extension Fabric development environment running, that can be used, too (i.e. if its one started by the extension recently). 
  
  Click on the ellipsis ("...") button under the **Fabric Environments** pane and "Start Fabric Runtime." Check your **Output** terminal pane at the bottom; you should see a message indicating that you've successfully submitted the proposal to join the channel. If this is the case, you can proceed.
  
2. Under the **Fabric Environments** pane, expand the **Smart Contracts** twisty and click on the **+ Install** and select the smart contract packaged earlier. You should soon see a message indicating it was installed on the local peer (in the lower right).
  
3. Next, choose the `papercontract` version 0.0.1 (see popup prompt) as the contract to install. You should then get a message in "Output" indicating that it was successfully installed.
  
4. Under the sidebar panel **Fabric Environments**, click on **+ Instantiate** and choose to instantiate the contract `papercontract@0.0.1` that you installed in the previous step. 
  
5. Paste in the string `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call, and hit ENTER.
  
6. When prompted to enter optional arguments, hit ENTER to leave it blank (there are no arguments). Accept the defaults for optional subsequent parameter(s) by hitting ENTER.
  
  After a minute or so, you should see a progress message in the bottom right indicating that it is instantiating. Check the output pane to see if it was successfully instantiated.

### Step 6: Create some blockchain identities and import them into the local Fabric wallet

Prior to executing the smart contract transactions below, you should create some identities so that the transactions can be submitted/signed by different transacting identities; as this is a merely a sample contract, you will use the existing Development Fabric CA to issue them. (In reality, both MagnetoCorp and Digibank would issue their own respective organisational identities for their respective apps.) Complete the following steps:

1. Under the **Fabric Environments** panel, locate "Nodes," expand it, highlight the CA node (for example, `ca.org1`), and right-click `.... Create Identity (....)`.

2. When prompted for the identity, provide the name "Isabella@MagnetoCorp" and hit ENTER. You should immediately see that an "Isabella@MagnetoCorp" wallet has been created under the local Fabric wallet in the "Fabric Wallets" pane at the bottom.

3. Next, perform Step 1 above once more, but this time create an identity for "Balaji@DigiBank" and check that it is listed in the local wallet under "Fabric Wallets." You'll use these identities to execute the transactions further down.

### Step 7. Execute the commercial paper smart contract transactions from client applications: MagnetoCorp and DigiBank

So far, you've installed and instantiated your smart contract on the blockchain. Now it's time to try out the smart contract transactions.

The commercial paper scenario describes contract transactions that are run by employees of two different organizations, MagnetoCorp and DigiBank. Using the IBM Blockchain Platform VS Code extension, you will execute the transactions in turn, connecting to the local Fabric Gateway, as each independent identity -- it's that easy to interact with your development blockchain network using different identities. (In the grander context, these identities would be consumed by the applications of the respective client organisations.) Figure 6 summarizes how they would interact using client applications and identities/wallets (provided to the employees of each company organization).

**Figure 6. "Papernet" -- overview of transaction flow**
![Transaction flow](images/flow-transaction.png)

#### Transaction 1: Execute an `issue` transaction as Isabella@MagnetoCorp

1. From the IBM Blockchain Platform VS Code sidebar panel, locate the **Fabric Gateways** sub-panel and click once on the local Fabric Gateway. When prompted, select the `Isabella@MagnetoCorp` identity to connect with.
  
2. Still under "Fabric Gateways', expand the "mychannel" twisty and then the "papercontract" twisty, in turn. You should see a list of transaction names, one of which is "issue."
  
3. Highlight the "issue" transaction and right-click "Submit Transaction." A pop-up window should appear at the top.
  
4. When prompted to enter parameters, copy and paste the following parameters (with double-quotes) **inside** the square brackets "[]" and hit ENTER, then hit ENTER again (to skip "transient data" entry):
  
  ```
  "MagnetoCorp","000010","2020-05-31","2020-11-30","5000000"
  ```
  
5. Check the message (in the **Output** pane) indicating that this transaction was successfully submitted.
  
6. Lastly, disconnect from the Gateway using the "disconnect" icon (from the "Fabric Gateways") sub-panel.

#### Transaction 2. Execute a `buy` transaction as Balaji@DigiBank

1. Click once on the `local Fabric` gateway and when prompted, choose the “Balaji@DigiBank” identity to connect with (and make sure it connects).
  
2. Still under "Fabric Gateways," expand the "mychannel" twisty and then the "papercontract" twisty, in turn.

3. Now highlight the "buy" transaction from the list of transactions and right-click "Submit Transaction." A pop-up window will appear.

4. When prompted to enter parameters, copy and paste the following parameters (including the double-quotes) **inside** the square brackets, `[]`, and hit ENTER, then hit ENTER again (to skip the "transient data" entry):
  
  ```
  "MagnetoCorp","000010","MagnetoCorp","DigiBank","4900000","2020-05-31"
  ```
  
5. Check the message (in the output pane) indicating that this transaction was successfully submitted.

#### Transaction 3. Execute a `redeem` transaction as Balaji@DigiBank -- six months later

The time has come in this commercial paper's lifecycle for the current owner (DigiBank) to redeem the commercial paper at face value and recoup the investment outlay. Typically, a client application script called `redeem.js` would perform this task from a client and related identity perspective. You can execute this transaction using the VS Code extension, using Balaji's certificate (from his wallet), as it is currently connected as a client identity.

1. Now highlight the `redeem` transaction from the list of transactions and right-click "Submit Transaction." A pop-up window will appear.
  
2. When prompted to enter parameters, copy and paste the following parameters (including the double-quotes) **inside** the square brackets, `[]`, and hit ENTER, then hit ENTER again (to skip the "transient data" entry):
  
  ```
  "MagnetoCorp","000010","DigiBank","2020-11-30"
  ```
  
3. Check the message (in the output pane) indicating that this redeem transaction was successfully submitted.

Well done! You've completed this tutorial and successfully interacted with the smart contract, which demonstrates a simple lifecycle of a commercial paper instance (with 3 transactions) on the blockchain.

## Summary

You've now learned how to deploy a simple yet substantial commercial paper smart contract sample to a Fabric blockchain network. You’ve seen how it can create, package, install, and instantiate a smart contract and use the IBM Blockchain Platform [VS Code extension](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) to submit transactions as different identities, which are recorded on the ledger. (Clearly, the extension provides a lot more -- such as the develop/debug/test lifecycle of a developer -- beyond the scope of this particular tutorial.)

[My next tutorial](https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension/) will concentrate on another application perspective, querying the ledger -- for example, getting the history of transactions for a particular asset. I'll answer questions like:

* What was the "paper" trail? (Get it?)
* Who performed the transactions (the identities involved)?
* Exactly when did they take place?
* What exactly were the changes made (i.e. the "deltas") for each transaction in that history?

This means adding query functionality to the smart contract, as well as some "getters" to get you the right information from the historical transactions. These results are sent back to the respective application clients.

To complete the next tutorials (Parts 2 and 3), you'll need to clone some sample artifacts (code, script files, etc.) from GitHub. To do this, open up a terminal window, locate your desired directory, and paste in the following commands:

```
cd $HOME
git clone https://github.com/mahoney1/commpaper
```

The repository should now be successfully cloned, in preparation for the next stage.

If you haven't done so, I recommend reading Horea Porutiu's excellent introductory tutorial, [Develop a smart contract with the IBM Blockchain Platform VS Code extension](https://developer.ibm.com/tutorials/ibm-blockchain-platform-vscode-smart-contract/), and try creating your own small starter smart contract.

Thanks for joining me!
