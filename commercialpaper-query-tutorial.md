# Adding Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension

[Commercial Paper History Report](pics/paperhistory.mp4)


## Introduction

In the [IBM Blockchain Platform VSCode Extension with Commercial Paper tutorial](https://github.ibm.com/IBMCode/Code-Tutorials/blob/master/run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension/index.md), we saw how an example of deploying and interacting with the Commercial Paper smart contract, in a scenario that tracks the lifecycle of a Commercial paper. But now we want to see the 'paper' trail of all activity that took place during its lifecycle - ie an immutable history of the asset: who did what, and when did it take place etc etc.

The aim of this tutorial, is to show the use of the IBM Blockchain Platform VSCode extension, to add query transaction functions to the Fabric Samples Commercial Paper smart contract, using the extension to upgrade the contract after adding query functionality. We'll provide the code changes in the tutorial by adding a new Query class (containing query functions) and with it, the means to query key/extract information from the ledger. We'll then show interaction with the smart contract from a client application in DigiBank. The goal is to get the history of transactions, for the lifecycle of a commercial paper instance, and display its history in a nicely formatted UI / HTML table, in a client browser. The tutorial is aimed at Developers, and exact instructions (in particular, editing source files to add provided code segments) are provided. Take time to see what's going on - you don't necessarily have to understand javascript in great detail for this !

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features under the covers - to complete these tasks.


## Background
There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment upon maturity.

The Commercial paper scenario (in the last tutorial) began with employees from MagnetoCorp and DigiBank transacting as participants from their respective organisations, to create an initial history. In this tutorial, we'll complete the lifecycle but adding a third investor, Hedgematic into the mix - purely to show more historical data (its quick and easy to create this history).


## Pre-requisites

1. You will need to have completed the [Commercial Paper tutorial](url), specifically, have version 0.0.1 of the Commercial paper smart contract package loaded in VSCode under your 'Smart Contract Packages' in the 'IBM Blockchain Platform' sidebar.

2. From a terminal window, go to the `basic-network` directory under the `$HOME/fabric-samples` (ie wherever you downloaded the cloned Github Fabric Samples repo), and run the following houskeeping scripts in sequence:

`./teardown.sh`

`docker volume prune`

`./start.sh`      

3. In VSCode, click on the IBM Blockchain Platform icon. You'll see version `0.0.1` of your smart contract packages listed up top. Go ahead and connect to your Fabric, via your running `myfabric` connection which should still be present in the 'Blockchain Connections' panel (from the previous tutorial), and `install` the smart contract package onto `peer0.org1.example.com` - then `instantiate` the contract by traversing your `myfabric` network, just as you had done in the previous tutorial ('myfabric....mychannel....right-click... Instantiate Smart contract'). This is the basis from which this tutorial will proceed.

A new clean Fabric and ledger is now available - from here, we will create the transaction Commercial Paper history, as part of our tutorial.

## Estimated time

After the prerequisites are completed, this should take approximately *60 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from Digibank - should be able to see the history (from the ledger) of a Commercial paper, now that it has been redeemed (some 6 months after it was initially issued). Luke (a developer@Digibank), needs to add query functionality to the smart contract, and provide the client apps for Digibank, so that Balaji (or indeed Isabella) can query the ledger from the application. The upgraded smart contract should be active on the channel so that the client applications can perform queries and report on the ledger history. 

OK, lets get started !

## Step 1. Add the main query transaction functions in papercontract.js

1.  In VSCode, have open, the folder with the smart contract completed in the previous tutorial

2.  Open the main contract script file `papercontract.js` under the `lib` folder - add the following lines as instructed below:

After the  `const { Contract, Context }` line (approx line 8)   add the following lines:

```// Tutorial specific require for reporting identity of transactor

const cryptoHash = require('crypto-hashing');

```
The reason to add this 'cryptoHash' line, is because we are creating a hash of the X509 certificate for the individual users that are actually submitting each transaction in our lifecycle. It is done for reporting purposes, when we render the invoking Identity information later on in the tutorial.

After the `// PaperNet specific classes ` line (approx line 16) add another class as follows:

```const QueryUtils = require('./query.js');```

don't worry about any errors reported in the status bar for now. 

3. Right-click on the `lib` folder and `Create new file` - call it `query.js` and hit ENTER - you'll now have a blank file.

4. Paste in, the contents of the file `query.js` that you downloaded in Step 5 of the previous tutorial - namely the `git clone https://github.com/mahoney1/commpaper` section. Hit CONTROL + S to save this file.

5. Switch back to `papercontract.js` and we'll add some more logic to the main contract script file. 

6. Find the function that begins `async issue` (approx line 70) and scroll down to the line `paper.setOwner(issuer);` and create a new line directly under (it aligns with the correct indentation in VSCode).

7. Now paste in the following code segment: This is to have a convenient way to report the true identity in queries later on.

```
// Add the creator hash, to the Paper state
        let hashId = await this.idGen(ctx);
        paper.setCreator(hashId);
```
This code is also before the line `await ctx.paperList.addPaper(paper);` in the `issue` function.

8. Repeat the paste (of the 3 line code segment above)  - in the `async buy` and `async redeem` functions - paste the 3 lines near the end of EACH of those functions - and BEFORE the following line shown below - that is, in each function:

`await ctx.paperList.updatePaper(paper);`

9. In the `async buy` function only - at line 120 (approx) in the code, beginning with the comment `// Check paper is not already REDEEMED` and add a line below the line `paper.setOwner(newOwner);` and inside the `isTrading()` branch       :

`paper.setPrice(price);`

10.  Next, add the following code segment, containing 3 functions (incl 2 query transaction functions), directly AFTER the CLOSING curly bracket of the `redeem` function and BEFORE - the last CLOSING bracket in the file `papercontract.js` (ie its ensuing `module.exports` declaration) . These 2 main query functions call the 'worker' query functions / iterators in the file `query.js`):

```
    /**
    * generate an Id hash of the transactor cert
    * @param {Context} ctx the transaction context
    */

    async idGen(ctx)     {

        // Add the creator hash, to the Paper record
        let creator = await ctx.stub.getCreator();
        //console.log('creator object is: ' + creator.toString());
        let pem = creator.getIdBytes().toString('utf8');
        let buffer = new Buffer(pem);
        let hashId = cryptoHash('hash256', buffer).toString('hex');
        return hashId;
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
    
 Note that once you've pasted this into VSCode, the `ESLinter` may report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane at the bottom by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). Once you've completed the formatting task, you can hit CONTROL + S to save your file. 
 
11. We have two more small functions to add - inside `paper.js`. Open the file `paper.js` under the `lib` directory in your VSCode session.
 
12. After the existing `setOwner(newOwner)` function  (at approx. line 40) under the description called `//basic setters and getters` section - add the following functions:

```
    setCreator(creator) {
        this.creator = creator;
    }
    setPrice(price) {
        this.price = price;
    }

```

Next hit CONTROL + S to save the file.

## Step 2. Implement 'worker' Query class  utility functions into your VSCode project - new file: query.js 

1. Create a new file under the `contract/lib` folder using VSCode - call it `query.js`

2. Copy the contents of the file `query.js` from the Github repo `github.com/mahoney1/commpaper` that you cloned earlier

3. Paste the contents into your VSCode session - you should have all the copied contents in your new query javascript worker file.

OK - lets move on to getting this new contract functionality, out on the blockchain to replace the older smart contract edition!

## Step 3. Upgrade our Smart Contract version using IBP VSCode Extension, Instantiate new version

1. We now need to add some changes to the `package.json` file - ie add a dependency name, and change the version in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.2" 
  - add the following line immediately under the "dependencies" section:     `"crypto-hashing": "^1.0.0",`    (including the 'comma')
  - hit CONTROL and S to save it.

2. Click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our smart contract, using the IBP VSCode extension. 

3. Firstly, package the contract - click on the `IBM Blockchain Platform` sidebar icon and under 'Smart Contract Packages' choose to 'Add new package' icon ('+')  and you'll see that version '0.0.2' becomes the latest edition of `papercontract'.

4. Next, install the contract itself: expand the 'Blockchain Connections' pane, under the channel `mychannel` choose `peer0.org1.example.com` and right-click...Install new contract, installing version "0.0.2" from the list presented up top. You should get a message it was successfully installed.

5. Now we can carry out the upgrade. Expand the 'Blockchain Connections' pane, under the channel `mychannel` choose `peer0.org1.example.com` and expand the `papercontract` twisty then highlight the current `papercontract@0.0.1` instance ....then ... right-click...'Upgrade smart contract', and choose "papercontract@0.0.2" from the list presented up top. 

  - Enter `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call ; 
  - Hit 'ENTER' - ie leave blank - when prompted to enter arguments
  
You should get a message in the console that the upgrade is taking place.

The upgrade will be executed, albeit it will take a minute (please note, as it has to build the new smart contract container) or so to show as the active contract, when listed under active containers docker ps. The container will have the contract version as a suffix.

[Upgrade Smart Contract](pics/upgrade.png)


## Step 4. Create a new Digibank query client app, to invoke query transactions

1. In VSCode, click on the menu option 'File....open Folder' and open the folder under `organization/digibank/application` and hit ENTER

2. Right-click on the folder in the left pane and create a new file `queryapp.js` then paste the contents of the file `queryapp.js` located in the `commercial-paper` repo that you copied from Step 5 in the previous Commercial Paper tutorial.

3. Once pasted, you can open choose 'View....Problems' to see the formatting/indentation errors - in the Problem pane, do a right-click `Fix all auto-fixable errors` and it should automatically fix all the indentation issues. 

4. Hit CONTROL and S to save the file, then click on the `Source Control` icon to commit the file, with a commit message. The `queryapp.js` client contains two query functions:

    - a `queryHist` function that gets the history of a Commercial paper instance and 
    - a `queryOwner` function that gets the list of Commercial Papers owned by an organization (provided as a parameter to the query function).

Next up, we'll test the new application client from a terminal window in Digibank's application folder (it does not matter whether we test from Magnetocorp or DigiBank in this example - we should say the same data on the ledger from either application client :-).


## Step 5. Perform transactions 'issue', 'buy' and 'redeem' to update the ledger

Lets create some transactions, which will have new invoking transactor info (for each transaction) we added in our code earlier. Note we've stood up a new `basic-network` so we will have a new ledger.  The sequence is:

1. Issue a paper as 'MagnetoCorp'
2. Buy the paper as 'Digibank' - the new owner
3. Buy the paper as 'Hedgematic' - changed owner 
4. Redeem the paper at face value - as 'Hedgematic' - with MagnetoCorp as the original issuer

Note that you will have installed any NodeJS dependencies for the client applications, as a set of steps in the previous tutorial. We will also make use of the identity wallets previously populated in that tutorial.

### Transaction #1: Execute an `issue` transaction as Isabella@MagnetoCorp

1. Open a terminal window, and change directory to MagnetoCorp's application directory:

`cd $HOME/fabric-samples/commercial-paper/organization/magnetocorp/application`

2. Now execute the first Commercial paper transaction from the `application` directory - the 'issue' transaction:

`node issue.js`

You should get messages confirming it was successful:

[Issue message](/pics/issue-output.png)

### Transaction #2: Execute a `buy` transaction as Balaji@DigiBank

1. In the same terminal window, change directory to DigiBank's application directory:

`cd ../../digibank/application`

2. Now execute the first Commercial Paper  'buy' transaction from the `application` directory:

`node buy.js`

You should get messages confirming it was successful:

[Issue message](/pics/buy-output.png)

### Transaction #3: Execute another `buy` transaction as Bart@Hedgematic

DigiBank are re-structuring their investment portfolio, and have decided to sell on the commercial paper for a small profit to release funds earlier. The purchaser is 'Hedgematic' who see this as an opportunity to increase their commercial paper portfolio and recouping the face value later on. Let's execute this transaction as an employee of Hedgematic.

1. Copy the  `buy2.js` client application script from the `commpaper` repo directory into the current `commercial-paper/organization/digibank/application` directory for now (make sure to insert the dot '.' in the command below):

`cp $HOME/commpaper/buy2.js . `

2. Copy the 'bart@hedgematic' wallet zip file from the `commpaper` previously cloned `commpaper` Github repo,  and save it into the `/tmp` directory (not your application directory) and extract it in '/tmp' - after extraction, you will have a directory `/tmp/wallet/bart@hedgematic` containing `Bart@Hedgematic's` identity wallet.

`cp $HOME/commpaper/wallet.zip /tmp`
`cd /tmp`
`unzip wallet.zip`

3. Now run the 2nd buy transaction (its using Bart's identity) as follows:

`node buy2.js`


### Transaction #4: Execute a `redeem` transaction as Bart@Hedgematic - six months later

The time has come, in this Commercial Paper's lifecycle, for the Commercial paper to be redeemed by its current owner (Hedgematic), and at face value, so it recoups its investment outlay. There is a client application, called `redeem.js` which will perform this task, and it needs to use `bart@hedgematic's` identity to perform it (currently the `redeem.js` sample script uses `balaji's` identity, but because Hedgematic have since bought the paper from Digibank, we need to modify it to redeem it properly as Hedgematic's Bart !)

1. Once again from a terminal window, and the same directory `$HOME/fabric-samples/commercial-paper/organization/digibank/application` - edit the file `redeem.js`

2. Change line 25 approx., beginning with `const wallet =` to read as follows (you may prefer to copy the line, and comment the original using `//` ) - the wallet points to the downloaded wallet directory as shown below:

`const wallet = new FileSystemWallet('/tmp/wallet');`   

(Or if you prefer, you can issue your own identity, using the currently active CA server, using the Fabric-CA utilities or APIs and change this JS script as appropriate).

3. Change line 38 approx beginning with `const userName =` to read as follows (you may prefer to copy the existing line, and comment the original line, using `//` ) - so the userName points to bart, the employee of Hedgematic:

`const userName = 'bart@hedgematic';`

(Or if you prefer, you can issue your own identity, using the currently active CA server, using the Fabric-CA utilities or APIs and change this JS script as appropriate).

4. Finally, change line 67 approx beginning with `const redeemResponse` and change the FOURTH parameter to become 'Hedgematic' :

` const redeemResponse = await contract.submitTransaction('redeem', 'MagnetoCorp', '00001', 'Hedgematic', '2020-11-30')`

All good - save your file (CONTROL + S) and commit any changes.


5. Now run the `redeem.js` script :

`node redeem.js`

You should get messages confirming it was successful:

[Issue message](docs/pics/redeem-output.png)

## Step 6. Launch the sample Digibank Client query application

1. From a terminal window, change directory to the `$HOME/fabric-samples/commercial-paper/organization/digibank/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the results from both the `queryHist` function and `queryOwner` functions in the terminal window. It also creates a file `results.json` for the `queryHist` part.

## Step 7. Display the formatted results to a browser app

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install any code or client per se, we just need to use a simple HTML file - it uses online CSS formatting and it performs a local `XMLHttpRequest() GET REST API` call to load the results (from the JSON file, avoiding CORS issues) and render it in the table. That `index.html` file is also in the `commpaper` Github repo that was cloned previously, please take time to peruse it. INFO: Note that this HTML file is provided 'as-is' and purely for the purposes of rendering in a FIREFOX browser (at the time of writing, some of the javascript (it doesn't use jquery by the way) formatting does not work in Chrome, but works fine in Firefox).

1. In a terminal windows, open the `application` directory if not already in that directory. Copy the contents of the file `index.html` from the `commpaper` repo into it. If you examine the HTML file in VScode Explorer, it performs an REST /GET API call and load a results file called `results.json` (created by the queries invoked earlier) and render these in a table in a browser. The `results.json` file contains the query results from the query you ran earlier.

2. Launch a Firefox browser session (see note earlier), providing the `index.html` file provided as a parameter - tested with Firefox eg.

`firefox index.html`

3. You should see the results in tabular form in the browser - expand or contract column widths as it suits, such as longer columns like `Invoking ID` etc - note that `TxId` here, is the Fabric transaction Id. The `Invoking ID` is a hash of the signer certificate that was used to perform each transaction (eg, original issue, buy, a further purchase by a different investment bank, then a final redeem etc). The identity hash would easily be mapped to a real identity in a corporate database (eg Magnetocorp) like an LDAP or Active Directory, ie for reporting purposes (who are the real transacting employees). Obviously, transactions originate from other organisation(s) too: information about the invoker from another 'other organization' could be resolved/displayed with other attributes as appropriate.

[Commercial Paper History Report](pics/history-report.png)

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion


You learned how to deploy a substantial Commercial Paper smart contract sample in the earlier tutorial, and here you have learned how to add queries/upgrade your contract using the IBM Blockchain Platform VSCode extension and used it to implement features from  Hyperledger Fabric's new programming model.  Take time to peruse and look at the transaction (query) functions in both `papercontract.js` and indeed the query utility functions, the Query class file `query.js` under the `lib` directory. Finally, you've shown how to render the results - such as the history of a Commercial Paper - in a simple browser-based HTML tabulated application.

Thank you for completing this!
