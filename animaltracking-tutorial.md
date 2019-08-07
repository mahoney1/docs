---

# Front matter (metadata).

abstract: "Develop, integrate and deploy your smart contract, and integrate with a simple React-based dashboard app"

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

completed_date: "2019-08-08"

components:
  - "hyperledger-fabric"
  - "hyperledger"
  - "IBM Blockchain Platform"
  - "IBM Blockchain Platform VS Code extension"

draft: false

excerpt: "Develop, integrate and deploy your smart contract, and integrate ledger history and events into a React-based app"

meta_description: "Learn how to develop/deploy a Typescript Smart Contract using IBM Blockchain VS Code Extension, invoke transactins/queries and render the results in a React Dashboard App"

meta_keywords: "animal tracking, queries, smart contract, IBM Blockchain, IBM blockchain platform, VS Code extension, typescript, Hyperledger Fabric, React"

last_updated: "2019-08-08"

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
  - title: "Part 1: Run a Commercial paper smart contract with the IBM Blockchain VS Code extension" 
    url: "https://developer.ibm.com/series/blockchain-running-enhancing-commercial-paper-smart-contract/"
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

subtitle: "Deploy a Typescript Smart Contract locally, then to IBM Cloud SaaS and integrate events/ledger data with a locally deployed React-based client Dashboard app"

tags:
  - "animaltracking" "typescript" "react" "government" "dashboard" "blockchain" "ibm blockchain" "ibm blockchain platform" "ibm blockchain platform vscode extension" "ibp" "ibp2" "SaaS"

title: "Use the IBM Blockchain Platform VS Code extension to deploy a Smart Contract locally, then promote to IBM Blockchain SaaS, and integrate events and ledger data with a sample React-based client dashboard app"

type: tutorial

---


## Introduction

This hands-on tutorial shows how to integrate query data from a blockchain ledger and events emitted by a smart contract instantiated on a channel, into a client-side React Dashboard app. It uses the [IBM Blockchain Platform VS Code extension](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) as the developer platform to manage the smart contract and clients - and essentially orchestrates the activity in this tutorial. The smart contract and the client apps, both written in Typescript, make use of the new features of the [new Hyperledger Fabric programming model](https://hyperledger-fabric.readthedocs.io/en/release-1.4/developapps/developing_applications.html), available since Hyperledger Fabric v1.4. 

The IBM Blockchain Platform VS Code developer extension is used to interact with two different environments - one local, and later on, IBM Blockchain Platform in the Cloud.

The tutorial flow takes you through deploying a TypeScript smart contract (using the new Fabric programming model) locally, and then promoting your contract to the Cloud. 

You will launch a locally installed React-based client dashboard, which will automatically render the source data (queries and emitted events) in the dashboard portal - the local Fabric environment is started in the IBP VS Code extension ; the Cloud instance uses the 'build a network' tutorial in the [IBM Blockchain Platform SaaS environment in IBM Cloud](https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network)

There are two sections: 

  - the first, **the development lifecycle**: deploy everything local ; use the IBM Blockchain Platform VS Code extension to manage the full IDE, stand up the runtime local Fabric and drive transactions, start an event listener. Then switch to the browser to show integration to the React client app; 
    
  - the second, **the promotion lifecycle**: package up the contract in IBP extension and promote it to the Cloud instance ie IBM Blockchain Platform SaaS environment - once again, use the IBP VS Code extension to drive the transaction flow, and again start a listener for emitted events from the Cloud. Once more, show your React App dashboard interacting with data from the IBM Cloud based ledger.
    
The tutorial provides the end-to-end steps to stand up a React-based dashboard client containing a fictitious 'animal co-operative' dashboard app, which providing summary charts, and ledger-based query and event data (emitted by the contract) sourced from the blockchain ledger (we're particularly interested in SHEEPGOAT (a Sheep/Goat cross-breed) related events :-) !). You'll use a Typescript-based animaltracking smart contract, and interact with it from Typescript application clients, and of course, the dashboard.

NOTE: All data that is rendered from the blockchain will have a 'lock' icon alongside the record for viewing purposes.

Figure 1. "Overview of React App Dashboard interacting with the Blockchain ledger for source data and events"


![AnimalTracking: Integrate summary, query data and events from the blockchain](img/highlevel-overvw.png)


The tutorial uses an 'intermediate' level, 'strongly typed', model-based Typescript contract and is aimed at Developers who wish to understand how to run this contract and then integrate resulting blockchain data (as returned to app clients) into a locally installed sample React JS based application, based on [Tabler UI React-based Dashboard](https://github.com/tabler/tabler-react). Take time to see what's going on - you don't necessarily have to understand Typescript, Javascript or React in great detail to understand this !  The lifecycle of typical transactions contained in the animal tracking contract is shown below. For more info on the very useful Tabler React project, you can check out the README and licence info [here](https://github.com/tabler/tabler-react/blob/master/README.md)

![Typical contract transaction lifecycle](img/animaltrackingcontract.png)

As mentioned, we're using IBM Blockchain Platform VS Code Extension - and the new Fabric programming model and SDK features under the covers - to complete these tasks. In particular, you will use Query and Event Typescript application clients - in addition to the IBP VS Code extension, to perform the required actions in this tutorial.

## Background

![General Flow with local Fabric or IBM Blockchain Platform SaaS edition](img/overview.png)


## Scenario

The use case centres around an Animal Co-operative dashboard, and the need to source the truth from the ledger for reporting purposes.

Jane Pearson has been at CONGA Co-op for 10 years now, and of late, she has taken on a very special role: she is responsible for keeping a handle on the SHEEPGOAT numbers (as a species their numbers needs growing). Data concerning their welfare or any incidents need to be monitored. Jane also needs key stats about SHEEPGOATS at her fingertips. While all data on the dashboard is important, part of her remit is to monitor info & events affecting SHEEPGOATS in the co-op region. In particular events that are critical, like SHEEPGOAT quarantines or ensuring vet inspections, as well as the less important new registrations.

Jane relies heavily on her dashboard app, from which she can instigate any inline actions against events as mentioned.

## Pre-requisites

1. You will need to have the following installed in order to proceed - it has been tested on a Linux virtual machine:

* (Node v8.x or greater and npm v5.x or greater)[https://nodejs.org/en/download/]
* (Docker version v17.06.2-ce or greater)[https://www.docker.com/get-started]
* (Docker Compose v1.14.0 or greater)[https://docs.docker.com/compose/install/]
* (VS Code)[https://code.visualstudio.com/download] editor and the IBM Blockchain VS Code extension — see the (marketplace)[https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform] for pre-requisites
* Install Yarn and Typescript (if not already installed) - see the 'Preparatory steps' to set the NPM prefix below

2. For Part 2 of this tutorial - deploying and integrating the dashboard App and smart contract to IBM Blockchain Platform SaaS - you will need to have an (IBM Blockchain Platform blockchain network)[https://cloud.ibm.com/catalog/services/blockchain] installed and running.

## Preparatory Steps

1.From a terminal window in your $HOME directory, install `typescript` (if not already installed) and `yarn` (ditto) as follows in your current user's HOME directory ('demo' in this case):

`npm config set prefix /home/demo`

`npm install typescript yarn`

`export PATH=$HOME/bin:$PATH`

2. Next, create a project directory (as a non-root user on Linux) called `dash`   eg assume `$HOME/dash` is the starting point 

`cd $HOME/dash` to enter that directory

3. Clone the `tabler-react` Github repository:

`git clone https://github.com/tabler/tabler-react.git`

4. Clone the `animaltracking` Github repository:

`git clone https://github.com/mahoney1/animaltracking.git`

5. IF you've previously deployed the `animal-tracking` smart contract, you are recommended to perform a `teardown` in the IBM Blockchain Platform VS Code extension (click on the icon) in VS Code - then select 'FABRIC OPS', click in the '...'  select 'TearDown Runtime Fabric' and confirm you want to tear down. After doing a teardown, start a new Fabric, again from 'FABRIC OPS' .....click on 'Start New Fabric' and ensure that you have a running, functional Fabric, and with the Nodes started, in the left sidepanel.

6. In the VS Code extension under 'Fabric Gateways', connect to your `local_fabric`  sidepanel, and use the`admin` identity to connect.

7. In VS Code Explorer, choose File > Open Folder, and navigate to the `animaltracking` folder in your cloned repo - then select the `contracts` folder, eg. navigating to the `$HOME/animaltracking/typescript/contract` directory. Depending on your VS Code version, you may be prompted to open the VS Code workspace provided in that directory. In any case, the `contract` folder must be your top-level project folder in VS Code Explorer before proceeding.

8. In the Terminal Window, run `npm install` to install the dependencies for the imported contract

9. Click on the IBM Blockchain Platform icon and from '...' ellipses on the 'Smart Contract Packages' panel, choose to 'Package a Smart Contract' - choose `animaltracking-ts@0.0.1`. You should get confirmation the package was successfully created.

10. Next, under 'Local Fabric Ops' choose to 'Install' the package onto the local peer - await a successful install message in VS Code.

11. Next, instantiate the `animaltracking-ts` Smart Contract by choosing 'Instantiate'  and when prompted, select `animaltracking-ts@0.0.1` as the contract to instantiate. 

    - When prompted to provide a function, supply the text:
      `org.example.animaltracking:instantiate` 
      and hit ENTER.
      
    - Hit ENTER to accept the remaining defaults for the remaining parameters when prompted. 
    
In approx. one minute or less, you should get confirmation the contract was successfully instantiated and you should see the instantiated contract called `animaltracking-ts@xxx`, under the 'Fabric Local Ops' pane.

12. Still in the VS Code extension, navigate to Nodes under 'Local Fabric Ops' and highlight `peer0.org1.example.com` and right-click....Export Connection profile and save it in your $HOME directory as filename `connection.json` - the client applications will use this.

13. From a terminal, navigate to the `animaltracking/typescript/client/lib` subdirectory. Edit the following Typescript files and replace the current the setting of the variable `currentDir` HOME directory (at approx line 28) to your own HOME directory setting assignment (currently it is set to '/home/demo') - to your own home directory ('/home/userxx') ie where your project is installed. 

`EventClient.ts`

'EventClient_IBP.ts`

`QueryClient.ts`

`QueryClient_IBP.ts`

`QueryEvents.ts`

next, run the following from the `client` subdirectory, to install client module dependencies then transpile the Typescript files to Javascript

`npm install`

`npm run build` 


14. For Part 2 of this tutorial, you will need an set up an (IBM Blockchain Platform SaaS instance)[https://cloud.ibm.com/catalog/services/blockchain-platform] and have completed the (Build a network tutorial)[https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network] 
 
Successful completion of these pre-reqs, is the basis from which this tutorial can now proceed. 

## Estimated time

After the preparatory steps are completed, this should take approximately *60 minutes* to complete.

OK, lets get started !

# PART 1: Integrate React Dashboard App, Blockchain Ledger data with a Local Fabric instance

## Steps


### Step 1. Install packages for React Dashboard using Yarn

1. Change directory to the cloned `tabler-react` directory and perform an install using the yarn package manager:

`cd $HOME/dash/tabler-react`

2. Edit the `package.json` file and change the devDependency for `eslint` to be `5.6.0` explicitly - then save the file.

`yarn install`

3. Backup some existing files in `tabler-react` - both in `src` and in the `example/src` subdirectories as follows:

`cd src`

`cp Tabler.css Tabler.css.bak`

`cd ../example/src`

`cp HomePage.react.js HomePage.react.js.bak`

`cp SiteWrapper.react.js SiteWrapper.react.js.bak`

(The above copies are merely so that you can do a `diff` of the changes made, to implement the animal tracking dashboard. Check out in particular, the `json` objects in `HomePage.react.js`, that represent JSON coming from the blockchain, via queries and events.

4. Now copy in/move the customisations from the `animaltracking` cloned repo as follows - **perform steps in the `example/src` subdirectory** in your Tabler React directory - ignore the warning (in the 1st copy command below) about being unable to copy a subdirectory, at this point.

`cp $HOME/dash/animaltracking/react/* .`

`mv Tabler.css ../../src`

`mkdir ../public/demo/icons`

`cp $HOME/dash/animaltracking/react/icons/* ../public/demo/icons`

5. Create a symbolic link to where our ledger data is persisted (for this tutorial, the React DashBoard App picks up data retrieved from the blockchain and persisted to a file)

`ln -s $HOME/dash/animaltracking/typescript/client/lib ledger`

6. Next, we need to install the dependencies/packages for the `example` application itself (up one directory level):

`cd ..`

`yarn install`

### Step 2. Start the React Dashboard app

1. Start the React dashboard app - change directory one level, to the `example` directory:

`yarn start`

2. You should get a browser launched with the Dashboard app active (when writing this tutorial, it launched Firefox on Ubuntu)


### Step 3. In a VS Code terminal window, start the Contract Event Listener

1. Still in VS Code, click on the `Terminal` tab at the bottom - press ENTER if prompted to `hit any key to close`. You should now have a command prompt - change directory to the animaltracking `client/lib` directory (from the `contract` subdirectory) and add execute permissions to some listener scripts as shown:

`cd ../client/lib`

`chmod +x listen*`

2. Start the local Event Listener by running the bash script `listenLocal.sh`:

`./listenLocal.sh`

You should get messages ("Getting Listener" etc) that its started and you should also see an event from the earlier `setupdemo` transaction (during setup, an event for a SHEEPGOAT registration was emitted).

### Step 4. In IBP VS Code extension, pre-populate the ledger with demo data

1.  In the IBP VS Code extension, connect to the 'Local Fabric' as `admin and expand the tree to reveal the `animaltracking-ts@xxx` contract under 'Fabric Gateways'. You will see a list of transactions. Scroll down to the end and you will find a transaction called `setupdemo`. We'll use this to pre-populate the ledger with some sample data.

2. Right-click on `setupdemo` and select `Submit Transaction` - accept all the default prompts for parameters (don't need to enter anything) etc - we won't need to supply any parameters.

### Step 5. Invoke Transactions in sequence to register further events

1. Go back to the list of `animaltracking-ts` transactions under 'Fabric Gateways' and right-click on `register` ... `Submit Transaction`.  When prompted, paste the following parameter list - including the double-quotes "" : in between the two square brackets `[ ]` in the VS Code prompt: 

`"SHEEPGOAT 000011 24/07/2019 BOVIS_ARIES FARMER.JOHN AVONDALE.LOC1 ARRIVALF1 IN_FIELD WOOL false"`
 
2. Click on the adjacent Terminal in VS Code and check that we had an event reported by the Event Listener for id `000011` .

3. Go back to the transaction list - this time, right-click on `quarantine` ... `Submit Transaction` . When prompted: paste these 2 parameters (each separated by double-quotes), in between the square brackets at the parameters prompt:

`"SHEEPGOAT", "000011"`

4. Once again, check from the terminal window that you've got an `ISOLATION` event posted.

5. Lastly, select the transaction `assigninspection` ... `Submit Transaction` . When prompted, paste the following into the square brackets:

`"SHEEPGOAT", "000011", "VET00007"`

6. Once again, check the Event Listener pane - we should have an INSPECT event reported.

We now have 4 events: an initial registration (from setupdemo), a new SHEEPGOAT registration, and two lifecycle events for this SHEEPGOAT.

7. In the Terminal window with the running Event Listener, stop it using CONTROL and C to stop. You should see an `Events Processed` message after interruption. We now have a set of 4 events, in JSON, contained in a file called `events.json`

(Note: generally, these events would picked up by an application using either something like Sockets.io (more efficient) or AJAX (less efficient) calls from the React or node application using React to render the UI. But for the purposes of this tutorial, we will write the events (emitted by the contract) to a simple JSON file: it is this that the React Dashboard app picks up, to dynamically present the ledger data in the application.

8. We also want to query all SHEEPGOAT registrations from the ledger, and present this in the dashboard. To do this, we need to run the QueryClient script - still in the same `client/lib` subdirectory - as follows:

`node query.js`

The script performs a number of queries, some of which are fulfilled given the ledger state at this point - however, the data we're interested in, will be the last query performed - and the results are written to a file called `registrations.json` and which our Dashboard will automatically pick up. One thing to note: registration ID `000011` will not show in the list in the latest registrations table: that's because (in this sample contract), the last function performed on that registration is one of assignment to a Vet.

### Step 6. Check the React Dashboard App for new Query and Events

Many of the charts/area diagrams contain summarised data can can be derived from the ledger, whether aggregated, calculated or indeed - in the case of temperatures trends - data that is aggregated from IoT temperatures over a period of time - this is just an example.

The data we're interested is 'new registrations' and separately, a 'Blockchain Events' table.

1. Switch to your browser launched earlier - all the data/records we're interested in contains a grey/blue 'lock' icon

2. Check that you have a list of new SHEEPGOAT registrations - these are the list of registrations extracted by the Query client earlier.

3. Scroll down to 'Recent Blockchain Events' - these represent our 4 blockchain events emitted by the `animaltracking-ts` Contract instantiated on channel `mychannel` and which were picked up by our Contract Listener earlier.

Well done! You've now completed this first part of the tutorial.

# PART 2: Integrate React Dashboard App, Blockchain Ledger data from an IBM Blockchain Platform SaaS instance

Note: If you wish to complete this part, and you are not already signed up for an IBP Cloud instance, you will need to do so (here)[https://cloud.ibm.com/catalog/services/blockchain-platform]. 

The tutorial use configuration metadata based on the ('Build your network tutorial')[https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network]) tutorial ie: 

MSP ID = 'org1msp'

Registered Identity = 'ibpuser'

Channel = "channel1"

When instantiating the animal tracking smart contract in the IBP environment, provide the following when prompted to enter a function for contract initialisation: 

Instantiate Parameter = "org.example.animaltracking:instantiate"

Contract Name = "animaltracking-ts";

![Creating the Blockchain Service)](img/welcomeback.png)


## Steps

### Step 1. Export CDS package and install/instantiate the contract using a peer Admin in IBP SaaS

We will need to package up our TypeScript smart contract into a (CDS file)[https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4noah.html#packaging] to deploy as a chaincode package for the IBP SaaS environment. We can do this easily from the IBM Blockchain Platform VS Code extension.

1. In VS Code, click on the IBP extension icon in the sidebar, and under 'Smart Contract Packages' select `animaltracking-ts@0.0.1` and right-click .... 'export Package' . Accept the default file name `animaltracking-ts@0.0.1.cds` and export it to a temporary directory on your filesystem. You should get a 'successful export' message popup at the bottom.

2. Login to your IBM Blockchain Platform Cloud instance and launch the IBM Blockchain Platform Console, close the welcome banner if required.

3. Click on the 'Smart Contracts' tab in your IBP environment. Click on 'Install Smart Contract' and when prompted, upload your CDS package from earlier and click 'Install Smart Contract' - await the 'success' message.

![Upload and Install the Contract](img/installcontract.png)

4. Next, click on the 'Overflow' menu alongside the installed `animaltracking-ts` smart contract. On the side panel that opens, select the channel `channel1` to instantiate the smart contract on - click Next. 
  
5. Accept the default endorsement policy and select the Org MSP set up for your environment: `org1msp`

6. Next, select the Org1 peer which is already joined to the channel. Skip the 'private data collections' prompt and click Next to proceed to instantiation

7. Lastly, paste in the text  `org.example.animaltracking:instantiate` when prompted to enter an initialisation function. Then accept the defaults thereafter. Your contract should be successfully instantiated on the channel `channel` and appear in the instantiated contracts further down the same page.

### Step 2. Register an IBP identity and export wallet for use later

1. Click on the 'Org1 CA' node and click on the button to 'Register User' - register a user 'ibpuser' with an enrol secret of 'demo' - 'register user'. This user will be the identity we use to invoke transactions, run queries etc from the IBP VS Code extension later.

2. Next, click on the action list for 'ibpuser' and choose to 'Enrol' ... enter the id and secret you entered above and click Next

3. Enter a 'display name' of 'ibpuser' and click on 'Export Identity' - this will prompt you to save the identity to your local filesystem as a JSON file - we will use this later.

![Register and Enrol user ibpuser](img/registeruser.png)

### Step 3. Export the connection profile to use in the IBP VS Code extension as a Gateway Connection

1. From the 'Smart Contracts' panel, scroll down to the 'Instantiated Smart Contracts' section

2. Click on the Overflow menu and select 'Connect with SDK'

3. Enter `org1msp` for the MSP choice and 'Org1 CA' as the Certificate Authority of choice. Scroll down to the end and click on 'Download Connection Profile' - 

4. The exported file will be called something like `channel1_animaltracking-ts_profile.json` in your Downloads directrory - rename this connection JSON file locally as `connection_IBP.json` and move it to your $HOME directory. This directory location will be used later by the Query Client application script that will query the IBP SaaS `channel` based ledger FYI.

![Export SDK Connection Profile info](img/connect-with-SDK.png)

### Step 4. Setup the IBP Gateway and import the Wallet to connect to IBP SaaS environment

1. Back in the IBP VS Code panel in your local development environment, disconnect from any currently connected local gateways.

2. Under 'Fabric Gateways' click on the '+' icon to 'add a new Gateway' - give it a name of IBPGW when prompted and Browser for the file `connection_IBP.json` in your HOME directory, click Select and it should be added successfully.

3. Next, import the `ibpuser` identity which was exported as a JSON file earlier. Click on the '+' symbol under 'Fabric Wallets'

4. Select to 'Create a new wallet and add identity' and give it a name of `IBPWallet` . Next enter `ibpuser` in response to the 'name of the identity' prompt

5. Enter `org1msp` when prompted to enter an MSP ID and hit ENTER

6. Choose the 'Provide a JSON file' when prompted to choose a method for adding an identity and browser for the `ibpuser.json` file in your HOME directory - select this. You should see messages that it was imported successfully and it will appear on the list.

7. Now connect to the Gateway under the 'Fabric Gateways' panel and click on 'IBPGW' - select to connect with the user `ibpuser`. You will see that this user is now connected through the IBPGW gateway as highlighted in the graphic below. We are now ready to try out our integration from the React Dashboard app, all the way through to the SaaS environment. At this point of course, we have no ledger data in the Saas environment, so we will need to populate it and start creating some transactions.

![Connect to IBP SaaS environment](img/ibpuser-connect.png)

### Step 5. Execute transactions and test out integration of new events, queries from the IBP SaaS environment

Note: before invoking the listener client and query client apps below - you should verify that your IBP parameters (eg. MSP ID, channel name, etc) mentioned at the start of 'Part 2' above have been implemented in the scripts.

1. From the IBP VS Code extension, expand the channel `channel` and the `animaltracking-ts@0.0.1` instantiated contract to reveal the list of available transactions - scroll down to the `setupdemo` transaction.

![Transaction Functions from IBP](img/ibpsaas-txn-list.png)

2. Right-click on `setupdemo` and click 'Submit Transaction' and when prompted, supply a value of "IBP" inside the 'parameter' prompt -  hit ENTER to accept all the remaining default prompts - this will populate our ledger with some new registrations and which are prefixed 'IBP' - so we can see the data is different in our dashboard.

3. Click on the terminal window in VS Code and from the `animaltracking/typescript/client/lib` subdirectory: start our IBP event listener process from the command prompt as follows:

`./listenIBP.sh`

You'll get messages that it is started and we should see an event has already occurred from the `setupdemo` transaction where new registrations means an emitted event.

4. Go back to the list of `animaltracking-ts` transactions under 'Fabric Gateways' and right-click on `register` ... `Submit Transaction`.  When prompted, paste the following parameter list - including the double-quotes "" : in between the two square brackets `[ ]` in the VS Code prompt: 

`"SHEEPGOAT IBP-000099 24/07/2019 BOVIS_ARIES FARMER.JOHN AVONDALE.LOC1 ARRIVALF1 IN_FIELD WOOL false"`
 
5. Click on the adjacent Terminal in VS Code and check that we had an event reported by the Event Listener for id `000011` .

6. Go back to the transaction list - this time, right-click on `quarantine` ... `Submit Transaction` . When prompted: paste these 2 parameters (each separated by double-quotes), in between the square brackets at the parameters prompt:

`"SHEEPGOAT", "IBP-000099"`

7. Once again, check from the terminal window that you've got an `ISOLATION` event posted.

8. Lastly, select the transaction `assigninspection` ... `Submit Transaction` . When prompted, paste the following into the square brackets:

`"SHEEPGOAT", "IBP-000099", "VET00007"`

9. Once again, check the Event Listener pane - we should have an INSPECT event reported.

We now have 4 events: an initial registration (from setupdemo), a new SHEEPGOAT registration, and two lifecycle events for this SHEEPGOAT.

10. In the Terminal window with the running Event Listener, stop it using CONTROL and C to stop. You should see an `Events Processed` message after interruption. We now have a set of 4 IBP events (with very recent timestamps), in JSON, contained in a file called `events.json`

11. We also want to query all SHEEPGOAT registrations from the ledger, and present this in the dashboard. To do this, we need to run the QueryClient script as follows:

`cd $HOME/dash/animaltracking/typescript/client/lib`

`node query_IBP.js`

The script performs a number of queries, some of which are fulfilled given the IBP SaaS ledger state at this point - results are written to a file called `registrations.json` which is automatically detected/rendered by the React Dashboard App.


### Step 6. Check the React Dashboard App for new Query and Events

Once again, the data we're interested in our Dashboard app are the 'new registrations' and separately, the 'Blockchain Events' table.

1. Switch to your React Dashboard browser tab launched earlier - all the data/records we're interested in contains a grey/blue 'lock' icon

2. Check that the refreshed list of new SHEEPGOAT registrations - these are the list of registrations extracted by the IBP Query client above.

3. Scroll down to 'Recent Blockchain Events' - these represent our 4 IBP blockchain events (one from `setupdemo`, one from `register` and one each from `quarantine` and `assigninspection`) emitted by the `animaltracking-ts` Contract instantiated on channel `channel1` in IBP SaaS and which were picked up by our IBP Contract Listener above.

Well done! You've now completed this second and final section of the tutorial.

![React Dashboard with Ledger data](img/highlevel-overvw.png)

## Conclusion

This tutorial aimed to show a simple use case, of combining the new features of:

  - the IBM Blockchain VS Code extension to manage the whole development inside VS Code - a powerful and seamless IDE for the developer.
  - Fabric 1.4's new programming model (eg, application client SDK, gateways, wallets, event SDK enhancements, and of course the new Fabric Contract changes)
  - Typescript Contract and Client applications (Event Listeners, App Clients etc)
  - Tabler React (the Github project at https://github.com/tabler/tabler-react for a React-based Dashboard app template dashboard app
  - Adding customisations, suited to the use case / Ledger data being extracted
  - starting local, verifying from your IBM Blockchain IDE that everything works locally ; then promote the smart contract to an IBM Blockchain Platform Cloud instance, and show integration to that SaaS environment from your IBP Clients and to the React Dashboard app.
    
Thanks for trying it out! If you have any issues, please raise an issue on my Github project (click 'Issues') at https://github.com/mahoney1/animaltracking with full details of the problem you're facing - thanks!)

As a last step, it is good practice to close out your current folders in VS Code, in preparation for your next tutorial or project.

With thanks to Robert Thatcher, IBM Early Experience Programs for his valued reviews input and testing of the tutorial.
