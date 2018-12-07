# Enhance Query functionality: Add Commercial Paper Deltas History using the IBM Blockchain VSCode extension

[Commercial Paper Deltas Report](pics/paperhistory.mp4)


## Introduction

In the [Adding Query functionality to the Commercial Paper using IBM Blockchain VSCode extension tutorial](url), we saw how to add rich query functionality using the Extension, to enable us to query/generate a report for the lifecycle of a Commercial Paper instance. Show the immutable history is of course important, but what if we just want to know: 'what were the changes committed for each transaction in that history'?

The aim of this tutorial is to add the capability to pull just the 'deltas' from the blockchain and report them. In summary those steps are:
- add the required functionality to the smart contract
- upgrade our existing Smart Contract (with existing Query class from previous tutorial) using the IBM Blockchain Platform VSCode extension. 
- Once instantiated, we will use a `queryapp` client application to pull the results
- Lastly, render the deltas in a HTML table in a client browser, reporting deltas for the Commercial Paper instance in question. 

We'll provide the code changes as part of this, adding to the existing Query class and the client app to extract deltas - then interact with the smart contract from client applications (command line and browser - to render in HTML). 

The end goal is to show the history of changes/deltas for all transactions executed during the lifecycle of a commercial paper instance.

We'll be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features - to complete these tasks.


## Background

There is a fantastic description of the Commercial Paper use case scenario in the latest [Fabric Developing Applications]( https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs and the scenario depicted there makes fascinating reading.   In short,  its a way for large institutions/organisations to obtain funds, to meet short-term debt obligations - and a chance for investors to to get return on investment upon maturity.

The Commercial paper scenario uses employees transacting as participants from their respective organisations, MagnetoCorp and DigiBank to create an initial history. We'll extend the lifecycle of the commercial paper, by involving a third investor, Hedgematic - purely to show more historical data to report upon.


## Pre-requisites

1. You will need to have completed the [Part 2: Adding Query functionalitytutorial](url), specifically, have version 0.0.2 of the Commercial paper smart contract package loaded in VSCode - as part of that, you'll have created the transaction history or 'paper trail'.

2. In VSCode, click on the IBM Blockchain Platform icon. You should have version `0.0.2` of your smart contract packages. 


3. Connect to your Fabric in the VSCode extension, via the running `myfabric` connection, it should still be present in the 'Blockchain Connections' panel - the contract has already been instantiated.


## Estimated time

After the prerequisites are validated, this should take approximately *45 minutes* to complete.

## Scenario

Isabella, an employee of MagnetoCorp and investment trader Balaji from Digibank - should be able to see the history of deltas (from the ledger) of a Commercial paper instance, and report on them in a client browser. Luke (a developer@MagnetoCorp) needs to add enhanced query functionality to the smart contract, then upgrade it - all using the IBP VSCode extension - and furthermore, provide the client apps for MagnetoCorp, so that Isabella can run a `deltas` query transaction and report on those in a simple browser app. 

OK, lets get started !

## Step 1. Add the `getDeltas` main query transaction in the main contract source file

1.  In VSCode, have open, the folder with the smart contract completed in the previous tutorial

2.  Open the main contract script file `lib/papercontract.js` under the `lib` folder - add the following lines as instructed below:

BEFORE the LAST function `queryOwner`  and AFTER the function `queryHist` add the following lines - this is the 'main' transaction query, that's called from our client application:

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
You'll notice that it calls the existing `getHistory` function (in the Query class, in `query.js`) - this uses an iterator to get the full history of the paper. Next, it calls a function `getDeltas` (also from `query.js` - a function that doesn't exist - yet!). This latter function is responsible for resolving the 'deltas', after the paper was initially issued. We'll add this function in the next step.

Note that once you've pasted this into VSCode, the `ESLinter` may report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane at the bottom by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). 
 
Once you've completed the formatting task, you can hit CONTROL + S to save your file. 

## Step 2. Add the `getDeltas` query worker function to the Query class in `query.js`

1. Click on the source file `lib/query.js` - and add the 'worker' functions shown below, to get the deltas and also to return the data (to the calling query client app), in a JSONified form suitable for our `tabulator` HTML client app.

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

}

```


Note that once you've pasted this into VSCode, the `ESLinter` will again report a problem in the `Problems` pane. You can easily rectify the formatting issues by in the problems pane at the bottom by choosing `right-click....` then  `Fix all auto-fixable issues` - likewise, it will remove all trailing spaces if any are reported (ref. line number reported). 
 
2. Once you've completed the formatting task, and there are no more 'problems', you can hit CONTROL + S to save your file. 
 
Cool - lets move on to getting this new contract functionality, out on the blockchain to replace the older version !

## Step 3. Upgrade our Smart Contract version using IBP VScode Extension, Instantiate new edition

1. First, we need to update the dependencies and version for our contract. Update the `package.json` file - ie add a dependency name, and change the version in preparation for the contract upgrade. Click on the `package.json` file in Explorer, and:

  - change the `version` to "0.0.3" 
  - hit CONTROL and S to save it.

2. Next, click on the Source Control sidebar icon and click the `tick` icon to commit, with a message of 'adding queries' and hit ENTER.

We're now ready to upgrade our smart contract, using the IBP VSCode extension. 

3. Click on the `IBM Blockchain Platform` sidebar icon and under 'Smart Contract Packages' choose to 'Add new package' and you'll see that version '0.0.3' becomes the latest edition of `papercontract'.

4. Expand the 'Blockchain Connections' pane, under the channel `mychannel` choose `peer0.org1.example.com` and expand the `papercontract` twisty then highlight the current `papercontract@0.0.2` instance (that was deployed in the previous query tutorial) ....then ... right-click...'Upgrade smart contract', and choose "papercontract@0.0.3" from the list presented up top. 

  - Enter `org.papernet.commercialpaper:instantiate` when prompted to enter a function name to call ; 
  - Hit 'ENTER' - ie leave blank - when prompted to enter arguments
  
You should get a message in the console that the upgrade is taking place.

The upgrade will be executed, albeit it will take a minute (please note, as it has to build the new smart contract container) or so to show as the active contract, when listed under active containers `docker ps`. The container will have the contract version as a suffix.

[Upgrade Smart Contract](pics/upgrade.png)


## Step 4. Upgrade the MagnetoCorp query client app, to invoke a `queryDeltas` transaction

1. In VSCode, click on the menu option 'File....open Folder' and open the folder under `organization/magnetocorp/application` and hit ENTER

2. Open the file `queryapp.js`,  then paste the contents shown below, BEFORE the line that begins with the comment:

` // query the OWNER of a commercial paper`

Paste this code:
```
// query the DELTAS of the history for a commercial paper
        console.log('Calling queryDeltas to get the deltas of Commercial Paper instance 00001');
        console.log('========================================================================');
        // QUERY the deltas of a commercial paper providing it the Issuer/paper number combo below
        const deltaResponse = await contract.submitTransaction('queryDeltas', 'MagnetoCorp', '00001');
        // let queryresult = CommercialPaper.fromBuffer(queryResponse);

        let file2 = await fs.writeFileSync('deltas.json', deltaResponse, 'utf8');
        console.log('the deltas HISTORY response is ' + deltaResponse);
        console.log(' ');
```

3. Once pasted, you can open choose 'View....Problems' to see the formatting/indentation errors - in the Problem pane, do a right-click `Fix all auto-fixable errors` and it should automatically fix all the indentation issues. 

4. Hit CONTROL and S to save the file, then click on the `Source Control` icon to commit the file, with a commit message. The `queryapp.js` client contains two query functions:

    - a `queryHist` function that gets the history of a Commercial paper instance 
    - a `queryDeltas` function that gets the deltas of that history
    - a `queryOwner` function that gets the list of Commercial Papers owned by an organization (provided as a parameter to the query function).

Next up, we'll test the new application client from a terminal window.


## Step 5. Run the updated MagnetoCorp client query application

We already have a history of transactions at this point - from the previous tutorial and use the existing MagnetoCorp employee wallet to run the query:

1. From a terminal window, change directory to the `$HOME/fabric-samples/commercial-paper/organization/magnetocorp/application` folder

2. Run the queryapp client using node:

`node queryapp.js`

3. You should see the results from the `queryHist` function then the `queryDeltas` function and finally the `queryOwner` transaction in the terminal window. `queryHist` as we know creates a file `results.json` ; `queryDeltas` creates a file called `deltas.json`.

## Step 6. Display the formatted Deltas history results to a browser app

For this part, we'll use a simple Tabulator that will render our results in a nice HTML table. For more info on Tabulator, see http://tabulator.info/examples/4.1 . We don't have to install any code or client per se, we just need to use a simple HTML file - it uses online CSS formatting and it performs a local `XMLHttpRequest() GET REST API` call to load the results (from the JSON file, avoiding CORS issues) and render it in the table. That `index.html` file is also in the `commpaper` Github repo that was cloned previously, please take time to peruse it. INFO: Note that this HTML file is provided 'as-is' and purely for the purposes of rendering in a FIREFOX browser (at the time of writing, some of the javascript (doesn't use jquery by the way) formatting does not work in Chrome, but works fine in Firefox).

1. Launch a Firefox browser session (eg. install Firefox if you don't have it) providing the `index.html` file provided, along with the Issuer/Paper number as a parameter - eg.

`firefox deltas.html?myParam="MagnetoCorp:0001"`

2. You should see the deltas (what changed, in that transaction, by the invoking ID listed) in tabular form in the browser - expand or contract column widths as it suits, such as longer columns like `Invoking ID` etc. The `Invoking ID` is a hash of the signer certificate that was used to perform each transaction (eg, original issue, buy, a further purchase by a different investment bank, then a final redeem etc). The identity hash would easily be mapped to a real identity in a corporate database (eg Magnetocorp) like an LDAP or Active Directory, ie for reporting purposes (who are the real transacting employees). Obviously, transactions originate from other organisation(s) too: information about the invoker from another 'other organization' could be resolved/displayed with other attributes as appropriate.

Note also that we report the 'State' on each line (eg. two 'buy' transaction states in a row = not necessarily a change in 'State' per se,  but reported nonetheless for clarity)

[Commercial Paper History Report](pics/deltas-report.png)

Well done! You've completed the query tutorial for adding query functionality to the Commercial Paper sample smart contract using the IBM Blockchain Platform VSCode extension.

## Conclusion


You learned how to deploy a substantial Commercial Paper smart contract sample in the earlier tutorial, and here you have learned how to add queries/upgrade your contract using the IBM Blockchain Platform VSCode extension and used it to implement features from  Hyperledger Fabric's new programming model.  Take time to peruse and look at the transaction (query) functions in both `papercontract.js` and indeed the query utility functions, the Query class file `query.js` under the `lib` directory. Finally, you've shown how to render the results - such as the history of a Commercial Paper - in a simple browser-based HTML tabulated application.

Thank you for completing this!
