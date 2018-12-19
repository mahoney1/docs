---
# Front matter (metadata).

abstract: "Enhance Query functionality: Add Commercial Paper Deltas History using the IBM Blockchain VSCode extension"

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

completed_date: "2018-12-18"

components:
  - "hyperledger-fabric"
  - "hyperledger"

draft: false

excerpt: "Enhance Query functionality:  Add Deltas History to Commercial Paper sample, using the IBM Blockchain VSCode extension and report the history of changes"

meta_description: "Add further query functionality using IBM Blockchain VScode extension to the Commercial Paper sample, providing a history of asset deltas from an application client"

meta_keywords: "commercial paper, queries, smart contract, IBM blockchain, IBM blockchain platform, VSCode extension, Hyperledger Fabric"

last_updated: "2018-12-18"

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
  - title: "Part 2: Add Query functionality to the Commercial Paper Smart Contract with the IBM Blockchain VSCode Extension" 
    url: "url of final rendered https://github.com/mahoney1/docs/blob/master/commercial-paper-query-tutorial-meta.md"

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

subtitle: "Use IBM Blockchain Platform VSCode extension to upgrade an existing Commercial paper smart contract easily, and add further query capability"

tags:
  - "finance"

title: "Enhance Query functionality: Add Deltas History to Commercial Paper sample, using the IBM Blockchain VSCode extension"

type: tutorial

---

**Figure 1. "Papernet" -- getting the deltas (changes) only from the blockchain for a Commercial paper asset **
![Papernet: Asset history and lifecycle from a Blockchain query](images/delta-results.gif)

# Enhance Query functionality: Add Commercial Paper Deltas History using the IBM Blockchain VSCode extension


## Introduction

In the [Adding Query functionality to the Commercial Paper using IBM Blockchain VSCode extension tutorial](url-tba), we saw how to add rich query functionality and enhance our smart contract using the IBM Blockchain VSCode Extension. The tutorial enabled us to query the history and lifecycle of a Commercial Paper instance and report on it. Showing that immutable history is of course important, but what if we just want to know: 'what were the changes committed for each transaction in that history'? Such a use case is relevant, when dealing with large volumes of transactions, or looking for patterns etc. The example below demonstrates a smaller scale example, nevertheless, the principles - and indeed the code samples provided -apply to any ledger and use case.

The aim of this tutorial is to add the capability to pull just the 'deltas' from the blockchain and report on them. That is:

- add the required functionality for getting deltas only - to the existing smart contract logic
- upgrade the existing, query-rich Smart Contract (from previous tutorial) using the IBM Blockchain Platform VSCode extension. 
- Try it out with an updated DigiBank client application to pull the results - as an employee of 'DigiBank'
- Render the deltas in a HTML table in a client browser, reporting deltas for the Commercial Paper lifecycle in question - what changed, after it was initially created?. 

We'll provide the code blocks as part of this tutorial, adding to the existing Query class and also provide the client app changes to invoke a 'getDeltas' transaction. Finally, we'll display the results in a simple, HTML-based table. 

The goal is to show the history of changes/deltas ONLY (so - not ALL fields will show up in the HTML table, things that don't change,  will be 'blank' - save, some 'constants' like 'state' (eg two consecutive 'BUY' transactions by different organisations, are the same state)).

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features - to complete these tasks.


## Background

There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment upon maturity.

The previous Commercial paper scenario had users transacting as participants from their respective organisations, MagnetoCorp and DigiBank. We'll extend the lifecycle of the commercial paper, by involving another organisation, Hedgematic - mainly, to show more historical data in the lifecycle.


## Pre-requisites

1. You will need to have completed the tutorial called [Part 2: Adding Query functionality tutorial](https://github.com/mahoney1/docs/blob/master/commercial-paper-query-tutorial-meta.md)
2. Specifically, have `version 0.0.2` of the Commercial paper smart contract package loaded in IBM Blockchain Platform VSCode Extension (under 'Smart Contracts' pane) 
3. Have created the 'history' of transactions, ie created the transaction history or 'paper trail' from the tutorial mentioned.
4. Connect to your Fabric in the VSCode extension, via the running `myfabric` connection, it should still be present in the 'Blockchain Connections' panel - the contract has already been instantiated from 'previous'.
5. In VSCode Explorer, choose File > Open Folder, and select the `contracts` folder, by navigating to the `$HOME/fabric-samples/commercial-paper/organization/magnetocorp` directory. This must be your top-level project folder for this tutorial.



## Estimated time

After the prerequisites are validated, this should take approximately *45 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from Digibank - should be able to see the history of deltas specifically (as reflected by the ledger) of a Commercial paper instance, and report on them in a client browser. Luke (a developer@DigiBank) has been tasked with adding this enhanced query functionality to the smart contract, then upgrade it on the blockchain - all using the IBP VSCode extension. Furthermore, Luke must provide the client application too, so that Balaji can run a `queryDeltas` query transaction and report on those in a simple browser app. 

OK, lets get started !

## Steps

### Step 1. Add the `getDeltas` main query transaction in the main contract source file

1.  In VSCode, you have 'contracts' as your top level folder.

2.  Open the main contract script file `lib/papercontract.js` under the `lib` folder - add the following code block as instructed below:

BEFORE the LAST function `queryOwner` - and AFTER the function `queryHist` add the following lines - this is in the 'main' contract file, and the transaction `queryDeltas` below, will later be called from our client application:

```
/**
    * queryDeltas commercial paper
    * @param {Context} ctx the transaction context
    * @param {String} issuer commercial paper issuer
    * @param {Integer} paperNumber paper number for this issuer
    */
    async queryDeltas(ctx, issuer, paperNumber) {

    // Get a key to be used for History / Delta query
        let cpKey = CommercialPaper.makeKey([issuer, paperNumber]);
        let myObj = new QueryUtils(ctx, 'org.papernet.commercialpaperlist');
        let results = await myObj.getHistory(cpKey);
        let deltas = await myObj.getDeltas(results);
        let jsonstr = myObj.jsontabulate(deltas);
        return jsonstr;
    }

```
You'll notice that it calls the existing `getHistory` function (in the Query class, in `query.js`) - this uses an iterator to get the full history of the paper. Next, it calls a function `getDeltas` (also from `query.js` - a function that doesn't exist - yet!). This latter function is responsible for resolving the 'deltas', after the paper was initially issued. We'll add this 'utility' function in the next step.

3. Note that once you've pasted this into VSCode, the `ESLinter` (ie if it is enabled) may report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane at the bottom by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). 
 
4. Once you've completed the formatting task, save your file (choose Save from the menu or hit CONTROL and S as a shortcut, to save your file).

### Step 2. Add the `getDeltas` query worker function to the Query class in `query.js`

1. Click on the source file `lib/query.js` and open it - add the following 'worker' functions as instructed below - it will get the deltas and also to return the data (to the main `queryDeltas` function), in a JSONified form suitable for passing on to our `tabulator` HTML client app.

AFTER the existing `getHistory` function - and BEFORE the 'closing' brace (immediately before `module.exports` line) - paste the following two functions:

```
// =========================================================================================
    // getDeltas takes getHistory results for an asset and returns the deltas
    // =========================================================================================

    async getDeltas(obj)  {

        let deltaArr = [];
        let counter = 0;
        let xtra_checked;


        Object.getOwnPropertyNames(obj).forEach( function (key, idx) {
            xtra_checked=0;
            let stdfields = 'TxId, Timestamp, IsDelete';

            for (let field of Object.keys(obj[key]) ) {
                let val = obj[key];
                counter = idx+1;
                let val2 = obj[counter];

                if (counter < obj.length ) {

                    if ( (stdfields.indexOf(field)) > -1 ) {
                        deltaArr.push(field, val[field]);
                    }

                    if (field === 'Value') {

                        for (let element of Object.keys(val[field]) ) { // Value stanza
                            // changes: of value, existing field
                            if ( val2[field].hasOwnProperty(element) && (val[field][element] !==  val2[field][element] )) {
                                deltaArr.push(element, val2[field][element]);
                            }
                            // deletes: field/val deleted (! item.next))
                            if ( (!val2[field].hasOwnProperty(element)) )  {
                                deltaArr.push(element, val[field][element]);
                            }
                            // adds: (new in item.next),add once only!
                            if (!xtra_checked) {
                                for ( let xtra of Object.keys(val2[field]) ) {
                                //console.log("xtra is " + val2[field][xtra] + "checking field " + xtra + "bool " + val[field].hasOwnProperty(xtra) );
                                    if ( (!val[field].hasOwnProperty(xtra)) ) {
                                        deltaArr.push(xtra, val2[field][xtra]);
                                    }
                                }
                                xtra_checked=1;
                            } // if xtra
                        } // for each 'element' loop
                    } // if 'Value' in payload
                } // if less than obj.length
            } // for each 'field' loop
        }  // 'foreach' loop
        ); //closing Object.keys

        return deltaArr ;
    } // async getDeltas

    // =========================================================================================
    // jsontabulate takes getDelta results array and returns the deltas in tabulator(.info) form
    // rendered as a nicely formatted table in HTML
    // =========================================================================================

    jsontabulate(array)  {
        let i= 1;
        let length = array.length;
        let val = '[{'; // begins with - FYI below, each element is stripped of "" by key/value stepthru
        for (let [key, value] of Object.entries(array)) {
            console.log('key is' + key + 'value is ' + value);
            if ( i > 1 && ( (i % 2) === 0)  ) {

                if (i < length)  {
                    val = val + '"' + value + '"' + ',';}  // (output 2-tuple)
                else {
                    val = val + '"' + value + '"}]'; }  // last record
            }
            else {
                if (value === 'TxId') {val = val.replace(/,$/,'},{');} // delimit each record, just before TxId
                val = val + '"' + value + '"' + ':';   // key:value
            }
            i++;
        }
        return val;
    }

```


2. Note that once you've pasted this into VSCode, the `ESLinter` may (if enabled) report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane at the bottom by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (reference the line number reported). 
 
3. Once you've completed the formatting task, and ensuring there are no more 'problems' at the bottom, you can hit CONTROL and S to save your file. 
 
OK - lets move on to getting this new contract functionality, out on the blockchain to replace the older version !

### Step 3. Upgrade our Smart Contract version using IBP VScode Extension, Instantiate new edition

1. First, we need to update the version number for our contract. To do this, update the `package.json` file, and change the version in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.3" 
  - hit CONTROL and S to save it.

2. Next, click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our existing smart contract package, using the IBP VSCode extension. 

3. Click on the `IBM Blockchain Platform` sidebar icon and under 'Smart Contract Packages' choose to 'Add new package' and you'll see that version '0.0.3' becomes the latest edition of `papercontract'.

4. Expand the 'Blockchain Connections' pane, under the channel `mychannel` choose `peer0.org1.example.com` and expand the `papercontract` twisty then highlight the current `papercontract@0.0.2` instance (that was deployed in the previous query tutorial) ....then ... right-click...'Upgrade smart contract', and choose "papercontract@0.0.3" from the list presented up top. 

  - Enter `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call ; 
  - Hit 'ENTER' - ie leave blank - when prompted to enter arguments
  
**Figure 2. "Upgrading the 'papercontract' Smart Contract in IBM Blockchain VSCode Extension **
![Upgrading Smart Contract in VSCode](image/upgrade-contract.png)

You should get a message in the console that the upgrade is taking place.

The upgrade will be executed, it will take a minute or so (as it has to build the new smart contract container). You should momentarily get a 'successful instantiation' message popup at the bottom right. The container (when seen from `docker ps` will have the contract version (0.0.3) as a suffix FYI).



### Step 4. Upgrade the DigiBank query client app, to invoke a `queryDeltas` transaction

1. In VSCode, click on the menu option 'File....open Folder' and open the folder under `organization/digibank/application` and hit ENTER

2. Open the existing file `queryapp.js` (from the last tutorial),  then paste the contents shown below, BEFORE the line that begins with the comment - ` // query the OWNER of a commercial paper` :

Paste this code:

```
// query the DELTAS of the history for a commercial paper
console.log('Calling queryDeltas to get the deltas of Commercial Paper instance 00001');
console.log('========================================================================');

// QUERY the deltas of a commercial paper providing it the Issuer/paper number combo below
const deltaResponse = await contract.submitTransaction('queryDeltas', 'MagnetoCorp', '00001');

console.log('the deltas HISTORY response is ' + JSON.parse(deltaResponse));
// parse the response sent back from contract -> client app
let file2 = await fs.writeFileSync('deltas.json', JSON.parse(deltaResponse), 'utf8');
console.log(' ');
```

3. Once pasted, you can open choose 'View....Problems' to see the formatting/indentation errors - in the Problem pane, do a right-click `Fix all auto-fixable errors` and it should automatically fix all the indentation issues. 

4. Hit CONTROL and S to save the file, then click on the `Source Control` icon to commit the file, with a commit message. The `queryapp.js` client contains three query functions (two of which already existed):

    - a `queryHist` function that gets the history of a Commercial paper instance 
    - a `queryDeltas` function that gets the deltas of that history
    - a `queryOwner` function that gets the list of Commercial Papers owned by an organization (provided as a parameter to the query function).

Next up, we'll test the new application client from a terminal window.


### Step 5. Run the updated MagnetoCorp client query application

We already have a history of transactions at this point - from the previous tutorial. We'll now run the deltas query app, as Balaji from Digibank, using his existing wallet to run the query:

1. From a terminal window, change directory to the `$HOME/fabric-samples/commercial-paper/organization/digibank/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the results from the `queryHist` function, followed by the `queryDeltas` function and finally the `queryOwner` transaction in the terminal window. In that file, `queryHist` (from previous) creates a file `results.json` ; and the `queryDeltas` transaction creates a file called `deltas.json` in the local directory.

### Step 6. Display the formatted Deltas history results to a browser app

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install any code or client per se, we just need to use a simple HTML file - it uses online CSS formatting and it performs a local `XMLHttpRequest() GET REST API` call to load the results (from the JSON file, avoiding CORS issues) and render it in the table. That `index.html` file is also in the `commpaper` Github repo that was cloned previously, please take time to peruse it. INFO: Note that this HTML file is provided 'as-is' and purely for the purposes of rendering in a FIREFOX browser (at the time of writing, some of the javascript formatting (forEach loop) does not work in Chrome, but it works fine in Firefox).

1. Launch a Firefox browser session (eg. install Firefox if you don't have it) providing the `index.html` file provided, along with the Issuer/Paper number as a parameter - eg.

`firefox deltas.html?myParam="MagnetoCorp:0001"`

2. You should see the deltas (what changed, in that transaction, by the invoking ID listed) in tabular form in the browser - expand or contract column widths as it suits. Note that the TxId here, is the Fabric transaction Id. The Invoking ID is the invoker Common Name, which is extracted using the Client Identity library from the previous Query tutorial. 

The deltas represent, what changed, after the asset was initially created (ie the 'issue' transaction) -  those are mainly, the owner of the asset, and the price paid (if more fields changed, obviously, the report would reflect that too).

Note also that we report the 'State' on each line (eg. two 'buy' transaction states in a row = not necessarily a change in 'State',  but reported nonetheless for clarity)

**Figure 3. "History of changes during the sample Commercial Paper lifecycle **
![History of changes during the sample Commercial Paper lifecycle](image/deltas-report.png)

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion


You learned how to enhance your existing smart contract, using the IBM Blockchain Platform to orchestrate and deploy the changes in your development environment. You've added code to capture just the deltas from a full asset history and rendered these in a HTML-based client application.

Thank you for completing this!
