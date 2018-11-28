# Adding Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension

## Introduction

In the [IBM Blockchain Platform VSCode Extension with Commercial Paper tutorial](url), we saw how an example of interacting with the Commercial Paper smart contract and the scenario that tracks the lifecycle of a Commercial paper. But now we want to see the 'paper' trail of all activity that took place during its lifecycle - ie the immutable history of the asset, who did what, and when did it take place etc etc.

The aim of this tutorial, is to add query transaction functions to the Fabric Samples Commercial Paper smart contract, upgrading the contract to add query functionality, as well as some more history. We'll provide the code changes in the tutorial, and show interaction with the upgraded smart contract, from client applications. The end goal is to trace the history of transactions executed during the lifecycle of a commercial paper instance.

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features - to complete these tasks.


## Background
There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment.

The scenario uses employees transacting (queries in this case) as participants from their respective organisations, MagnetoCorp and DigiBank. We'll extend the lifecycle of the commercial paper, by involving a third investor, Hedgematic - purely to show more historical data.


## Pre-requisites

1. You will need to have completed the [Commercial Paper tutorial](url) 

2. From the command line, specifically in the `basic-network` directory under the `$HOME/fabric-samples` (ie wherever you downloaded the cloned Github repo), run the following houskeeping scripts in sequence.

`./teardown.sh`

`docker volume prune`

`./start.sh`      

3. In VSCode, click on the IBM Blockchain Platform icon. You'll see version `0.0.1` of your smart contract packages. Go ahead and connect to your Fabric, via your running `myfabric` connection (check with `docker ps` if required), and `install` the smart contract package onto `peer0.org1.example.com` - then instantiate the contract, just as you had done in the previous tutorial. This is the basis from which this tutorial will proceed.

A new clean Fabric and ledger is now available - from here, we will create the transaction Commercial Paper history, as part of our tutorial.

## Estimated time

After the prerequisites are installed, this should take approximately *45 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from Digibank - should be able to see the history (from the ledger) of a Commercial paper, now that it has been redeemed (some 6 months after it was initially issued). Luke (a developer@MagnetoCorp)needs to add query functionality to the smart contract, and provide the client apps for MagnetoCorp, so that Isabella (MagnetoCorp) can query the ledger from the application (and likewise - for DigiBank application users of course!). The upgraded smart contract should be active on the channel so that the client applications can perform queries and report on the ledger history. 

OK, lets get started !

## Step 1. Add the main query transaction functions in papercontract.js

1.  In VSCode, have open, the folder with the smart contract completed in the previous tutorial

2.  Open the main contract script file `papercontract.js` - add the following lines as instructed below:

After the  `const { Contract, Context }` line (approx line 8)   add the following lines:

```// Tutorial specific require for reporting identity of transactor

const cryptoHash = require('crypto-hashing');```

```
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

8. Repeat the paste (of the 3 line code segment above)  - in the `async buy` and `async redeem` functions - paste the 3 lines near the end of EACH of those functions, and before the following line shown in each function:

`await ctx.paperList.updatePaper(paper);`


9.  Finally, add the following code segment, containing 3 functions (incl 2 query transaction functions), directly AFTER the CLOSING curly bracket of the `redeem` function and BEFORE - the last CLOSING bracket in the file `papercontract.js` (ie its ensuing `module.exports` declaration) . These 2 main query functions call the 'worker' query functions / iterators in the file `query.js`):

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

        // Get a key to be used for the paper, and get this from world state
        let cpKey = CommercialPaper.makeKey([issuer, paperNumber]);
        let myObj = new QueryUtils(ctx, 'org.papernet.commercialpaperlist');
        let results = await myObj.getHistory(cpKey);
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
    
 Note that once you've pasted this into VSCode, the ESLinter may report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane by `right-click....` then select `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). Once you've completed the formatting task, you can hit CONTROL + S to save your file. 
 
10. We have one more small function to add. Open the file `paper.js` under the `lib` directory in your VSCode session.
 
11. After the `setOwner(newOwner)` line (approx line 40) under the 'basic setters and getters` - add the following function:

```
    setCreator(creator) {
        this.creator = creator;
    }
```
Next hit CONTROL + S to save the file.

## Step 2. Implement 'worker' Query class  utility functions into your project - new file: query.js 

1. Create a new file under the `contract/lib` folder using VSCode - call it `query.js`

2. Copy the contents of the file `query.js` from the Github repo `github.com/mahoney1/commpaper` that you cloned earlier

3. Paste the contents into your VSCode session - you should have all the copied contents in your new query javascript worker file.

Cool - lets move on to getting this new contract functionality, out on the blockchain to replace the older version !

## Step 3. Upgrade our Smart Contract version using IBP VScode Extension, Instantiate new edition

1. We now need to add some changes to the `package.json` file - ie add a dependency name, and change the version in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.2" 
  - add the following line immediately under the "dependencies" section:     `"crypto-hashing": "^1.0.0",`    (including the 'comma')
  - hit CONTROL and S to save it.

2. Click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our smart contract, using the IBP VSCode extension. 

3. Click on the `IBM Blockchain Platform` sidebar icon and under 'Smart Contract Packages' choose to 'Add new package' and you'll see that version '0.0.2' becomes the latest edition of `papercontract'.

4. Expand the 'Blockchain Connections' pane, under the channel `mychannel` choose `peer0.org1.example.com` and right-click...Install new contract, installing version "0.0.2" from the list presented up top. You should get a message it was successfully installed.

5. Right-click on the channel `mychannel` - and choose the option to Instantiate/Upgrade Smart Contract, choose the "0.0.2" smart contract package to upgrade on the channel. 
  - Enter `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call ; 
  - Hit 'ENTER' - ie leave blank - when prompted to enter arguments

The upgrade will be executed, albeit it will take a minute or so to show as the active contract, when listed under active containers `docker ps`. The container will have the contract version as a suffix.

[Upgrade Smart Contract](pics/upgrade.png)


## Step 4. Create a new MagnetoCorp query client app, to invoke query transactions

1. In VSCode, click on the menu option 'File....open Folder' and open the folder under `organization/magnetocorp/application` and hit ENTER

2. Right-click on the folder in the left pane and create a new file `queryapp.js` then paste the contents of the file `queryapp.js` in the `commercial-paper` repo copied from Step 5 in the previous tutorial.

3. Once pasted, you can open choose 'View....Problems' to see the formatting/indentation errors - in the Problem pane, do a right-click `Fix all auto-fixable errors` and it should automatically fix all the indentation issues. 

4. Hit CONTROL and S to save the file, then click on the `Source Control` icon to commit the file, with a commit message. The `queryapp.js` client contains two query functions:

    - a `queryHist` function that gets the history of a Commercial paper instance and 
    - a `queryOwner` function that gets the list of Commercial Papers owned by an organization (provided as a parameter to the query function).

Next up, we'll test the new application client form a terminal window.


## Step 5. Perform transactions 'issue', 'buy' and 'redeem' to update the ledger

Lets create some transactions, which will have new invoking transactor info (for each transaction) we added in our code earlier. Note we've stood up a new `basic-network` so we will have a new ledger.  The sequence is:

1. Issue a paper as 'MagnetoCorp'
2. Buy the paper as 'Digibank' - the new owner
3. Buy the paper as 'Hedgematic' - changed owner 
4. Redeem the face value as 'Hedgematic' with MagnetoCorp as the issuer

Note that you will have installed any NodeJS dependencies for the client applications, as a set of steps in the previous tutorial. We will also make use of the identity wallets previously populated in that tutorial.

### Transaction #1: Execute an `issue` transaction as Isabella@MagnetoCorp

1. Change directory to MagnetoCorp's application directory:

`cd commercial-paper/organization/magnetocorp/application`

2. Now execute the first Commercial paper transaction from the `application` directory - the 'issue' transaction:

`node issue.js`

You should get messages confirming it was successful:

[Issue message](/pics/issue-output.png)

### Transaction #2: Execute a `buy` transaction as Balaji@DigiBank

1. Change directory to DigiBank's application directory:

`cd commercial-paper/organization/digibank/application`

2. Now execute the first Commercial Paper  'buy' transaction from the `application` directory:

`node buy.js`

You should get messages confirming it was successful:

[Issue message](/pics/buy-output.png)

### Transaction #3: Execute another `buy` transaction as Bart@Hedgematic

1. We will need to get the identity credentials for 'bart' that are provided in the cloned 'commpaper` repo directory. Copy the 'bart' folder to a new directory called '/tmp/wallet/'  - in the example below, the Github repo was previously cloned to the $HOME directory:

`mkdir /tmp/wallet`
`cp -r $HOME/commmpaper/bart /tmp/wallet`

2. copy the `buy2.js` client application script from the `commpaper` repo directory to the current `commercial-paper/organization/digibank/application` ` directory for now (make sure to insert the '.' in the command below):

`cp $HOME/commpaper/buy2.js .

3. Copy the 'bart@hedgematic' wallet zip file to the `/tmp` directory and extract it - you will have a directory `/tmp/wallet/bart@hedgematic` containing `Bart@Hedgematic's` digital identity

3. Run the 2nd buy transaction (using Bart's identity) as follows:

`node buy2.js`


### Transaction #4: Execute a `redeem` transaction as Balaji@DigiBank - six months later

The time has come, in this Commercial Paper's lifecycle, for the Commercial paper to be redeemed by its owner (Digibank), at face value, and recoup the investment outlay. There is a client application, called `redeem.js` which will perform this task, and its it needs to use `bart@hedgematic` identity to perform it (currently the `redeem.js` sample script uses `balaji's` identity, but because Hedgematic bought the paper from Digibank, we need to redeem it as Hedgematic !

1. From the same directory `commercial-paper/organization/digibank/application` - edit the file `redeem.js`

2. Change line 25 approx beginning with `const wallet =` to read as follows (you may prefer to copy the line, and comment the original using `//` ) - the wallet points to the downloaded wallet directory.

`const wallet = new FileSystemWallet('/tmp/wallet');`   

3. Change line 38 approx beginning with `const userName =` to read as follows (you may prefer to copy the line, and comment he original using `//` ) - the userName points to bart, the employee of Hedgematic

`const userName = 'bart@hedgematic';`

4. Finally, change line 67 approx beginning with `const redeemResponse` and change the FOURTH parameter to 'Hedgematic' :

` const redeemResponse = await contract.submitTransaction('redeem', 'MagnetoCorp', '00001', 'Hedgematic', '2020-11-30')

All good - save your file (CONTROL + S) and commit any changes.


5. Now run the `redeem.js` script 

`node redeem.js`

You should get messages confirming it was successful:

[Issue message](docs/pics/redeem-output.png)

## Step 6. Launch the sample MagnetoCorp Client query application

1. From a terminal window, change directory to the `commercial-paper/organization/magnetocorp/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the results from both the `queryHist` function and `queryOwner` functions in the terminal window. 

## Step 6. Display the formatted results to a browser app

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install a client per se, we just need to provide a simple HTML file that performs an `XMLHttpRequest() GET REST API` call to load the results (from a JSON file) and render it in the table. The HTML file is also in the `commpaper` Github repo that was cloned previously.

1. In a terminal windows, open the `application` directory if not already there. Copy the file `index.html` into it, from the `commpaper` Github repo that was cloned previously. If you examine the HTML file in VScode Explorer, it will perform an REST API call and load a results file called `results.json` (created by the queries invoked earlier) and render these in a table in a browser. The `results.json` contains the output, of the query response from the `node queryapp.js` call earlier.

2. Launch a browser (eg. Firefox) providing the `index.html` file provided as a parameter eg.

`firefox index.html`

3. You should see the results in tabular form in the browser - select to expand or contract columns as you wish, eg the `TxId` is the Fabric transaction Id. The `Invoked ID` is a hash of the signer certificate used to perform transactions previously (eg, issue, buy, a further purchase by a different investment bank, then a final redeem etc). The identity hash would easily be mapped to a real identity  in a corporate database (eg Magnetocorp) like an LDAP or Active Directory, ie for reporting purposes (who are the real transacting employees). Obviously, a transaction will originate from other organisation(s) too: information about the invoker from another 'other organization' could be resolved/displayed with other attributes as appropriate..

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion


You learned how to deploy a very substantial Commercial Paper smart contract sample in the earlier tutorial, and learned how to add queries using the IBM Blockchain IDE and using Hyperledger Fabric's newest programming model.  Take time to peruse and look at the transaction (query) functions in both `papercontract.js` and indeed the query utility functions the Query class file `query.js` under the `lib` directory. Finally, you've shown how to render the results - such as the history of a Commercial Paper - in a simple browser-based HTML application.

Thank you for completing this!
