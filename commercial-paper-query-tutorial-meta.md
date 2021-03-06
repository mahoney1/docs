---
# Front matter (metadata).

abstract: "Learn how to add Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension, execute the queries from a client application and render the transaction history of the Commercial paper, in a nice browser-based UI."

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

completed_date: "2018-12-18"

components:
  - "hyperledger-fabric"
  - "hyperledger"

draft: false

excerpt: "Add Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension and render a full asset history in a browser-based UI application"

meta_description: "Learn how to add Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension and then execute the queries from client application, rendering them in Tabulator."

meta_keywords: "commercial paper, queries, smart contract, IBM blockchain, IBM blockchain platform, VSCode extension, Hyperledger Fabric"

last_updated: "2019-01-08"

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
  - title: "Part 1: Run a Commercial paper smart contract with the IBM Blockchain VSCode extension" 
    url: "https://developer.ibm.com/tutorials/run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension/"
  - title: "Video: Start developing with the IBM Blockchain Platform VSCode Extension"
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

subtitle: "Use IBM Blockchain Platform VSCode extension to upgrade an existing Commercial paper smart contract easily, and get the full lifecycle and history of a Commercial paper asset in a browser-based UI application"

tags:
  - "finance"

title: "Enhance and add queries to a commercial paper smart contract with the IBM Blockchain VSCode extension"

type: tutorial

---

**Figure 1. "Papernet" -- querying the blockchain for the history of a Commercial paper asset **
![Papernet: Asset history and lifecycle from a Blockchain query](images/final-results.gif)


## Introduction

In the [IBM Blockchain Platform VSCode Extension with Commercial Paper tutorial](https://github.ibm.com/IBMCode/Code-Tutorials/blob/master/run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension/index.md), we saw how an example of deploying and interacting with the Commercial Paper smart contract, in a scenario that tracks the lifecycle of a Commercial paper. But now we want to see the 'paper' trail of all activity that took place during its lifecycle - ie an immutable history of the asset: who did what, and when did it take place etc etc.

**Figure 1. "Papernet" -- overview of commercial paper history and organisations involved**

![Transaction flow](images/flow-overview.png)

The aim of this tutorial, is to show the use of the IBM Blockchain Platform VSCode extension, to add query transaction functions to the Fabric Samples Commercial Paper smart contract, using the extension to upgrade the contract after adding query functionality. We'll provide the code changes in the tutorial by adding a new Query class (containing query functions) and with it, the means to query key/extract information from the ledger. We'll then show interaction with the smart contract from a client application in DigiBank. The goal is to get the history of transactions, for the lifecycle of a commercial paper instance, and display its history in a nicely formatted UI / HTML table, in a client browser. The tutorial is aimed at Developers, and exact instructions (in particular, editing source files to add provided code blocks) are provided. Take time to see what's going on - you don't necessarily have to understand javascript in great detail for this !

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features under the covers - to complete these tasks.


## Background

There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment upon maturity.

The Commercial paper scenario (in the last tutorial) began with employees from MagnetoCorp and DigiBank transacting as participants from their respective organisations, to create an initial history. In this tutorial, we'll complete the lifecycle but adding a third investor, Hedgematic into the mix - purely to show more historical data (its quick and easy to create this history).


## Pre-requisites

1. You will need to have completed the [Commercial Paper tutorial](https://developer.ibm.com/tutorials/run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension/), specifically, have version 0.0.1 of the Commercial paper smart contract package loaded in VSCode under your 'Smart Contract Packages' in the 'IBM Blockchain Platform' sidebar.

2. From a terminal window, go to the `basic-network` directory under the `$HOME/fabric-samples` (ie wherever you downloaded the cloned Github Fabric Samples repository), and run the following houskeeping commands/scripts in sequence:

`cd $HOME/fabric-samples/basic-network`

`./teardown.sh`

`docker volume prune` (answer 'y' when prompted)

`docker network prune` (answer 'y' when prompted)

`./start.sh`      

3. Go back into VSCode, then click on the IBM Blockchain Platform icon. You should see version `0.0.1` of your smart contract packages listed up top (this is from the previous tutorial). Go ahead and connect to your 'fresh' Fabric, via your running `myfabric` connection which should still be present in the 'Blockchain Connections' panel (from the previous tutorial), and `install` the smart contract package onto `peer0.org1.example.com` - then `instantiate` the contract by traversing your `myfabric` network, just as you had done in the previous tutorial:

 - Under 'Blockchain connections' select ` 'myfabric....mychannel....right-click... Instantiate Smart contract' `). 
 - Supply `org.papernet.commercialpaper:instantiate` when prompted for 'what function to call'.

4. In VSCode Explorer, choose File > Open Folder, and select the `contracts` folder, by navigating to the `$HOME/fabric-samples/commercial-paper/organization/magnetocorp` directory. The `contracts` folder must be your top-level project folder in VSCode.

Completion of the pre-reqs, is the basis from which this tutorial will proceed. From here, we will complet the detailed transaction Commercial Paper history, as part of this tutorial.

## Estimated time

After the prerequisites are completed, this should take approximately *45-60 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from DigiBank - should be able to see the history (from the ledger) of a Commercial paper, now that it has been redeemed (some 6 months after it was initially issued). Luke (a developer@Digibank), needs to add query functionality to the smart contract, and provide the client apps for DigiBank, so that Balaji (or indeed Isabella) can query the ledger from the application. The upgraded smart contract should be active on the channel so that the client applications can perform queries and report on the ledger history. 

OK, lets get started !

## Steps

### Step 1. Add the main query transaction functions in papercontract.js

1.  In VSCode, open the folder (if not already open) `contract` , containing the smart contract completed in the previous tutorial.

**Figure 2. Open the commercial paper sample project in VSCode**
  ![Open the commercial paper sample project in VSCode](images/papercontract.png)
  
2.  Open the main contract script file `papercontract.js` under the `lib` folder - add the following lines as instructed below:

After the  `const { Contract, Context }` line (approx line 8),  add the following lines:

```
// Tutorial specific 'require' for reporting identity of transactor using the Client Identity Chaincode Library (CID)

const ClientIdentity = require('fabric-shim').ClientIdentity;
```

This library is to enable us to obtain the invoker or identity submitting each transaction in our commercial paper lifecycle. It is done for reporting purposes, when we render the invoking identity information in a browser app, later on in the tutorial.

3. After the `// PaperNet specific classes ` line (approx line 16) add another class declaration as follows:

```
const QueryUtils = require('./query.js');
```

don't worry about any errors reported under 'Problems' (if you have ESLint enabled) in the status bar for now. 


4. Still in `papercontract.js`, find the function that begins `async issue` (approx line 70) and scroll down to the line `paper.setOwner(issuer);` in the function - create a blank/new line directly under it (should align with the correct indentation in your code).

5. Now paste in the following code block: The following code enables us to report the true identity of the transaction. The function `idGen` below is a class based function in `papercontract.js` ,  that uses the Client Identity Chaincode Library (CID) to obtain the attribute containing the invoking id (from its X509 Certificate).

```
    // Add the invoking CN, to the Paper state
    let invokingId = await this.idGen(ctx);
    paper.setCreator(invokingId);
    
```
Note: this code should be located BEFORE the line `await ctx.paperList.addPaper(paper);` in the `issue` function.

6. Repeat the paste as shown above (ie the 3 line block)  - into the functions beginning with 
   - `async buy` and 
   - `async redeem` 
   
   Paste the 3 line code block, near the end of EACH of those functions - and BEFORE the following line shown in each function : `await ctx.paperList.updatePaper(paper);`

7. In the `async buy` function only - at line 120 (approx) in the code, eg line with comment `// Check paper is not already REDEEMED` -  ADD this single line of code, BELOW the line `paper.setOwner(newOwner);` but INSIDE the `isTrading()`  code 'branch':

```
paper.setPrice(price);
```

8.  Next, add the following code block, containing the 3 functions below (two of which are our 'callable' query transaction functions), directly AFTER the CLOSING curly bracket of the `redeem` transaction function,  but must be BEFORE - the last CLOSING bracket in the file `papercontract.js` (ie the one immediately before the `module.exports` declaration) . These 2 main query functions call the 'worker' query functions / iterators in the file `query.js` and the `idGen` function below gets identity information, used for reporting):

```
   /**
    * grab the invoking CN from the of the X509 transactor cert
    * @param {Context} ctx the transaction context
    */

    async idGen(ctx)     {
 
        // Use the Client Identity Chaincode Library (CID) to get the invoker info.
        let cid = new ClientIdentity(ctx.stub);
        let id = cid.getID(); // X509 Certificate invoker is in CN form
        let CN = id.substring(id.indexOf("CN=") + 3, id.lastIndexOf("::"));
        return CN;
    }


    /**
    * queryHist commercial paper
    * @param {Context} ctx the transaction context
    * @param {String} issuer commercial paper issuer
    * @param {Integer} paperNumber paper number for this issuer
    */
    async queryHist(ctx, issuer, paperNumber) {

        // Get a key to be used for History query
        let cpKey = CommercialPaper.makeKey([issuer, paperNumber]);
        let myObj = new QueryUtils(ctx, 'org.papernet.commercialpaperlist');
        let results = await myObj.getHistory(cpKey);
        //console.log('main: queryHist was called and returned ' + JSON.stringify(results) );
        return results;

    }


    /**
    * queryOwner commercial paper
    * @param {Context} ctx the transaction context
    * @param {String} issuer commercial paper issuer
    * @param {Integer} paperNumber paper number for this issuer
    */
    async queryOwner(ctx, owner, paperNumber) {

        // Get a key to be used for the paper, and get this from world state
        // let cpKey = CommercialPaper.makeKey([issuer, paperNumber]);
        let myObj = new QueryUtils(ctx, 'org.papernet.commercialpaperlist');
        let owner_results = await myObj.queryKeyByOwner(owner);
        
        return owner_results;
    } 
 
 ```
    
Note: once you've pasted this into VSCode, the `ESLinter` extension - ie if it is enabled in your VSCode session, may report problems in the `Problems` pane at the bottom. If so, you can easily rectify the formatting issues by in the problems pane by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported by ESLint (ie. the line number is reported). Once you've completed the formatting task, save your file, via the menu, save your edits ! (you can hit 'CONTROL and S' as a shortcut to save your file). FYI: The ESLint extension (also available from the VSCode extension marketplace) is a very useful too and I'd recommend using it to fix any indentation, incorrect pasting or general errors that can be detected before you package up the smart contract.
 
9. We have two more small functions to add - this time, inside the source file `paper.js`. Open the file `paper.js` under the `lib` directory in your VSCode session.
 
10. AFTER the existing `setOwner(newOwner)` function  (at approx. line 40) - and under the description called `//basic setters and getters`  - ADD the following code block (contains two functions):

```
    setCreator(creator) {
        this.creator = creator;
    }
    setPrice(price) {
        this.price = price;
    }

```

Next hit CONTROL + S to save the file. 

### Step 2. Add requisite 'worker' Query class functions into your VSCode project - New file: query.js 

1. Create a new file via the VSCode menu,  under the `contract/lib` folder using VSCode - call it `query.js`

2. Copy the contents of the file `query.js` from the Github repo `github.com/mahoney1/commpaper` that you cloned earlier.

3. Paste the contents into your `query.js` VSCode edit session - you should have all the copied contents in your new query javascript 'worker' `query.js` file. Go ahead and save this file ! This ends the smart contract edits.

Let's move on to getting this added smart contract functionality, out on the blockchain, to replace the older smart contract edition!

### Step 3. Upgrade our Smart Contract version using IBP VSCode Extension

1. We now need to add a version change to the `package.json` file in our project - in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.2" 
  - hit CONTROL and S to save it.

2. Click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our smart contract, using the IBP VSCode extension. 

3. Firstly, package the contract - click on the `IBM Blockchain Platform` sidebar icon and under 'Smart Contract Packages' choose to 'Add new package' icon ('+')  and you'll see that version '0.0.2' becomes the latest edition of available `papercontract' packages.

4. Next, upgrade the contract itself: expand the 'Blockchain Connections' pane below, under the channel `mychannel` choose `peer0.org1.example.com` . Expand the `peer0.org1.example.com` twisty and select the `papercontract@0.0.1` entry specifically.  (You will notice also, that all of our transaction functions are listed below (expand the 'twisty' to view), including the new query functions we added).

5. Now choose right-click on `papercontract@0.0.1`...'Upgrade Smart Contract', and choose "papercontract@0.0.2" from the list presented (up top). 

  - Enter or paste the text `org.papernet.commercialpaper:instantiate` when prompted to enter `a function name to call` and hit ENTER ; 
  - Hit 'ENTER' - ie leave blank - when prompted to enter arguments
 
 **Figure 3. Upgrade the smart contract using IBP VScode extension**
 ![Upgrade the smart contract using IBP VScode extension](images/upgrade-contract.png)
 
You should get a message in the console that the upgrade is taking place. It will take a minute or so (please note, as it has to build the new smart contract container) and will display a 'successful' pop message when it has completed. When checked using `docker ps` from a terminal, it will show a new docker container with the contract version as a suffix under the 'Names' column.


### Step 4. Create a new DigiBank query client app, to invoke query transactions

1. In VSCode, click on the menu option 'File....open Folder' and open the folder under `organization/digibank/application` and hit ENTER

2. Right-click on the folder in the left pane and create a new file `queryapp.js` , then paste the contents of the file `queryapp.js` located in the `commercial-paper` repo directory, that you copied from 'Step 5' (eg from `$HOME/commpaper/queryapp.js` ) in the previous Commercial Paper tutorial.

3. Once pasted, you fix any formatting errors, if ESLint is enabled (right-click `Fix all auto-fixable errors` and it should automatically fix all the indentation issues).

4. Hit CONTROL and S to save the file, then click on the `Source Control` icon to commit the file, with a commit message. The `queryapp.js` client contains two query functions:

    - a `queryHist` function that gets the history of a Commercial paper instance and 
    - a `queryOwner` function that gets the list of Commercial Papers owned by an organization (provided as a parameter to the query function).

Next up, we'll create the transaction history, then run the new query client app to execute a set of queries, from a terminal window (from a terminal window in DigiBank's application folder - it does not matter whether we test from Magnetocorp or DigiBank in this example - we should see the same data on the ledger from either application client :-)).


### Step 5. Perform transactions 'issue', 'buy' and 'redeem' to update the ledger

Lets create some transactions, invoked as different identities/invokees and update the world state/create a transaction history on the ledger.  The sequence is:

1. Issue a paper as 'MagnetoCorp'
2. Buy the paper as 'DigiBank' - the new owner
3. Buy the paper as 'Hedgematic' - changed owner 
4. Redeem the paper at face value - as existing owner 'Hedgematic' - with MagnetoCorp as the original issuer

Note that you will have installed any application NodeJS dependencies (`npm install`) for the existing client applications, from the previous tutorial. We will also make use of the existing identity wallets previously populated in that tutorial (these are located at the same level as `application` under both the `magnetocorp` and `digibank` subdirectories respectively.

**Figure 4. "Papernet" -- overview of transaction flow**

![Transaction flow](images/flow-transaction.png)


#### Transaction #1: Execute an `issue` transaction as Isabella@MagnetoCorp

1. Open a terminal window, and change directory to MagnetoCorp's application directory (assuming $HOME is the holding location below):

`cd $HOME/fabric-samples/commercial-paper/organization/magnetocorp/application`

2. Now execute the first Commercial paper transaction from the `application` directory - the 'issue' transaction:

`node issue.js`

You should get messages confirming it was successful:

**Figure 5. The "issue" transaction**
![The issue transaction](images/issue-output.png)

#### Transaction #2: Execute a `buy` transaction as Balaji@DigiBank

1. In the same terminal window, change directory to DigiBank's application directory:

`cd ../../digibank/application`

2. Now execute the first Commercial Paper  'buy' transaction from the `application` directory:

`node buy.js`

You should get messages confirming it was successful:

**Figure 6. The "buy" transaction**
![The "buy" transaction](images/buy-output.png)

#### Transaction #3: Execute another `buy` transaction as Bart@Hedgematic

DigiBank are re-structuring their investment portfolio, and have decided to sell on the commercial paper for a small profit to release funds earlier. The purchaser is 'Hedgematic' who see this as an opportunity to increase their commercial paper portfolio and seek to recoup the face value of the paper at some point in the future. Let's execute this transaction as an employee of Hedgematic. (For convenience, we've provided a temporary wallet for the Hedgematic employee 'Bart', so that Hedgematic can invoke a `buy` transaction.)

1. Copy the  `buy2.js` client application script from the `commpaper` repo directory into the current `commercial-paper/organization/digibank/application` directory for now (make sure to insert the dot '.' in the command below):

`cp $HOME/commpaper/buy2.js . `

2. Copy the file 'wallet.zip' from the `commpaper` repo directory, into the `/tmp` directory. Then unzip it, example shown below:


`cp $HOME/commpaper/wallet.zip /tmp`

`cd /tmp`

`unzip wallet.zip`

It will unzip the user bart's wallet into a subdirectory under `/tmp/wallet`. After extraction, you will have a directory `/tmp/wallet/bart@hedgematic` containing `Bart@Hedgematic's` identity wallet.

3. Now run the 2nd buy transaction (its using Bart's identity in the client application `buy2.js`) as follows:

`node buy2.js`

You should get messages confirming it was successful:

**Figure 7. The "2nd buy" transaction**
![The "2nd buy" transaction](images/buy-output2.png)

#### Transaction #4: Execute a `redeem` transaction as Bart@Hedgematic - six months later

The time has come, in this Commercial Paper's lifecycle, for the Commercial paper to be redeemed by its current owner (Hedgematic), and at face value, so it recoups its investment outlay. There is a client application, called `redeem.js` which will perform this task, and it needs to use `bart@hedgematic's` identity from owner Hegematic, to perform it (currently the `redeem.js` sample script uses `balaji's` identity, but because Hedgematic have since bought the paper from DigiBank, we need to modify the JS script to redeem it properly as Hedgematic's Bart !). For the purposes of this tutorial, we will simply run the client application script for `redeem` from the `digibank` application subdirectory.

1. Once again from a terminal window, and the same directory `$HOME/fabric-samples/commercial-paper/organization/digibank/application` - edit the file `redeem.js`

2. Change line 25 approx. - beginning with `const wallet =` , to read as follows: (you may prefer to copy the line, and comment the original using `//` - the wallet points to the downloaded wallet directory as shown below):

`const wallet = new FileSystemWallet('/tmp/wallet');`   

(Or if you prefer, you can issue your own identity, using the currently active CA server, using the Fabric-CA utilities or APIs and change this JS script as appropriate).

3. Change line 38 approx beginning with `const userName =` to read as follows (you may prefer to copy the existing line, and comment the original line, using `//` ) - so the userName points to bart, the employee of Hedgematic:

`const userName = 'bart@hedgematic';`

(Or if you prefer, you can issue your own identity, using the currently active CA server, using the Fabric-CA utilities or APIs and change this JS script as appropriate).

4. Finally, change line 67 approx beginning with `const redeemResponse` and change the FOURTH parameter to become 'Hedgematic' :

` const redeemResponse = await contract.submitTransaction('redeem', 'MagnetoCorp', '00001', 'Hedgematic', '2020-11-30')`

All good - save your file (CONTROL and S) and commit any changes.


5. Now run the `redeem.js` script :

`node redeem.js`

You should get messages confirming it was successful:

**Figure 8. The "redeem" transaction -- the last in the lifecycle**
![The "redeem" transaction](images/redeem-output.png)


### Step 6. Launch the sample DigiBank Client query application

1. From a terminal window, change directory to the `$HOME/fabric-samples/commercial-paper/organization/digibank/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the JSON results from both the `queryHist` function and `queryOwner` functions in the terminal window. It also creates a file  called `results.json` in the current directory (history of the asset) as a result of the `queryHist` query invocation.

**Figure 9. The "queryapp" client results **
![The "queryapp" client results](images/queryapp-results.png)

### Step 7. Display the History in a nice HTML-based UI

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install any code or client per se, nor use jquery - we just need to use a simple HTML file - it uses online CSS formatting and it performs a local `XMLHttpRequest() GET REST API` call to load the local results (from the JSON file, avoiding CORS issues) and render it in the table. That `index.html` file comes from the `commpaper` Github repo that was cloned previously, please take time to peruse the HTML file. INFO: Note that this HTML file is provided 'as-is' and purely for the purposes of rendering in a FIREFOX browser (at the time of writing, some of the javascript (it doesn't use jquery by the way) formatting does not work in Chrome (doesn't like forEach !), but has been tested in Firefox).

1. In a terminal window, open the digibank `application` directory once again (you should already be there). 

2. Copy the file `index.html` from the `commpaper` repo (cloned earlier) into it. Eg. (with trailing '.' below, for current dir):

`cp $HOME/commpaper/index.html . `

If you examine the HTML file in VScode Explorer, it performs a `REST /GET API` call and loads a results file called `results.json` (created by the queries app invoked earlier) and render these in a table in a browser. The `results.json` contains the query results.

3. Launch a Firefox browser session (see note earlier), providing the `index.html` file provided as a parameter - tested with Firefox eg.

`firefox index.html`

4. You should see the results in tabular form in the browser - expand or contract column widths as it suits, such as longer columns like `Invoking ID` etc - note that `TxId` here, is the Fabric transaction Id. The `Invoking ID` is the invoker Common Name, which is extracted using the Client Identity library mentioned earlier. Obviously, it would need to be made available as an attribute by any of the organisations executing transactions using this shared smart contract. . Alternatives may be a hash of the signer certificate that was used to perform each transaction.

**Figure 10. Commercial Paper: Asset History Report **
![Commercial Paper: Asset History Report](images/history-report.png)

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion

You have learned how to add queries and upgrade your existing Commercial paper contract using the IBM Blockchain Platform VSCode extension, and use features from Hyperledger Fabric's new programming model.  Take time to peruse the transaction (query) functions in both `papercontract.js` and the Query class file `query.js` under the `lib` directory. Also, take time to peruse the client application `queryapp.js`.

Finally, you've shown how to render the history results - ie history of a Commercial Paper asset - in a simple browser-based HTML application. The final tutorial (which is based on this), will be to show only 'deltas' or changes during the lifecycle.

As a last step, it is good practice to close out your current folders in VSCode, in preparation for the next tutorial - adding smart contract functionality to query only the 'deltas' for the history of a particular asset - this will be the 3rd tutorial in the series

Thank you for completing this! 
