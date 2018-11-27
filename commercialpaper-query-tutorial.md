# Adding Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension

## Introduction

In the [Commercial Paper tutorial](url), we saw how to execute transactions that typify the lifecycle of a commercial paper lifecycle. 

The aim of this tutorial, is to add query transaction functions to the Commercial Paper smart contract, redeploy (upgrade) the modified contract, and show interaction from client applications. One example of this, is to trace the history of transactions executed during the lifecycle of the commercial paper.

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features - to complete these tasks.


## Background
There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment.

The scenario uses employees transacting (queries in this case) as participants from their respective organisations, MagnetoCorp and DigiBank. We'll extend the lifecycle of the commercial paper, by involving a third investor, Hedgematic - purely to show more historical data.


## Pre-requisites

You will need to have completed the [Commercial Paper tutorial](url) and the transactions from that tutorial should be present, as this tutorial builds on the contract created previously

## Estimated time

After the prerequisites are installed, this should take approximately *45 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from Digibank - should be able to see the history from the ledger. This required Luke (a developer@MagnetoCorp) to add query functionality in the contract, and then develop client apps for MagnetoCorp specifically, so that Isabella can query the ledger from an application (and likewise for DigiBank application users). The upgraded smart contract should be active on the channel so that the applications can perform queries and report on the ledger history. 

OK, lets get started !

## Step 1. Add the main query transaction functions in papercontract.js

1.  In VSCode, open the folder with the smart contract completed in the previous tutorial:

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

```// Add the creator hash (created from the identity's cert), to the Paper record
        let creator = await ctx.stub.getCreator();
        //console.log('Transaction creator is: ' + creator.toString());
        let pem = creator.getIdBytes().toString('utf8');
        let buffer = new Buffer(pem);
        let hashId = cryptoHash('hash256', buffer).toString('hex');

        paper.setCreator(hashId); // this is a function defined in `paper.js` which you'll add later FYI
```

Repeat this 8 line copy - in the `buy` and `redeem` functions - paste the 8 lines near the end of EACH of those functions, before the following line shown in each function:

`await ctx.paperList.updatePaper(paper);`


8.  Finally, add the following 2 query transaction functions / code segment, directly AFTER the `redeem` function ( after its closing bracket) and BEFORE - the last closing bracket in `papercontract.js` (and its ensuing `module.exports` declaration)  in `papercontract.js` . These 2 main functions call the 'worker' query functions / iterators in the file `query.js`):

```
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
    
 Note that once you've pasted this into VSCode, the ESLinter may report a problem in the Problems pane. You can easily rectify the formatting issues by in the problems pane by `right-click....` then select `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). Once you've completed the formatting task, you can hit CONTROL + S to save your file. 
 
9. We have one more small function to add. Open the file `paper.js` under the `lib` directory in your VSCode session.
 
10. After the `setOwner(newOwner)` line (approx line 40) under the 'basic setters and getters` - add the following function:

```
    setCreator(creator) {
        this.creator = creator;
    }
```
Next hit CONTROL + S to save the file.

## Step 2. Implement 'worker' query utility functions into your project - file: query.js 

1. Create a new file under the `contract/lib` folder using VSCode - call it `query.js`

2. Copy the contents of the file `query.js` from the Github repo `github.com/mahoney1/commpaper` that you cloned earlier

3. Paste the contents into your VSCode session - you should have all the copied contents in your new query javascript worker file.

Cool - lets move on to getting this new contract functionality, out on the blockchain to replace the older version !

## Step 3. Upgrade our Smart Contract version using IBM Blockchain Platform VScode Extension,  and Instantiate new edition

1. We now need to add some changes to the `package.json` file - ie add a dependency name, and change the version in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.2" 
  - add the following line immediately under the "dependencies" section:     `"crypto-hashing": "^1.0.0",`  
  - hit CONTROL and S to save it.

2. Click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our smart contract, from within VSCode itself. 

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


## Step 5. Launch the sample MagnetoCorp Client query application

1. From a terminal window, change directory to the `commercial-paper/organization/magnetocorp/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the results from both the `queryHist` function and `queryOwner` functions in the terminal window. 

## Step 6. Display the formatted results to a browser app

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install a client per se, we just need to provide a simple HTML file that performs an `XMLHttpRequest() GET REST API` call to load the results (from a JSON file) and render it in the table. The HTML file is also in the `commpaper` Github repo that was cloned previously.

1. In a terminal windows, open the `application` directory if not already there. Copy the file `index.html` into it, from the `commpaper` Github repo that was cloned previously. If you examine the HTML file in VScode Explorer, it will perform an REST API call and load a results file called `results.json` (created by the queries invoked earlier) and render these in a table in a browser. The `results.json` contains the output, of the query response from the `node queryapp.js` call earlier.

2. Launch a browser (eg. Firefox) providing the `index.html` file provided as a parameter eg.

`firefox index.html`

3. You should see the results in tabular form in the browser - select to expand or contract columns as you wish, eg the `TxId` is the Fabric transaction Id. The `Invoked ID` is a hash of the signer certificate used to perform transactions previously (eg, issue, buy, redeem etc). You'll notice that the `hash` function is only been implemented in the `issue` function for now and hence the IDs reported look the same - its easy to add this to the `buy`, `redeem` transactions too. Ordinarily, given that this is a client application (written by an organisation eg Magnetocorp ), the hash would easily be mapped to a real identity  in a corporate database like LDAP or Active Directory, ie for reporting purposes (and make sense of real transactors from their organization). Obviously, a transaction will originate from other organisation(s) too: information about the invoker from another 'other organization' could be resolved/displayed with other attributes as appropriate..

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion


You learned how to deploy a very substantial Commercial Paper smart contract sample in the earlier tutorial, and learned how to add queries using the IBM Blockchain IDE and using Hyperledger Fabric's newest programming model.  Take time to peruse and look at the transaction (query) functions in both `papercontract.js` and indeed the query utility functions the Query class file `query.js` under the `lib` directory. Finally, you've shown how to render the results - such as the history of a Commercial Paper - in a simple browser-based HTML application.

Thank you for completing this!
