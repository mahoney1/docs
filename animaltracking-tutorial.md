---

# Front matter (metadata).

abstract: "Develop, integrate and deploy your smart contract, and integrate with a simple React-based dashboard app"

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

completed_date: "2018-07-18"

components:
  - "hyperledger-fabric"
  - "hyperledger"
  - "IBM Blockchain Platform"
  - "IBM Blockchain Platform VSCode extension"

draft: false

excerpt: "Develop, integrate and deploy your smart contract, and integrate ledger history and events into a React-based app"

meta_description: "Learn how to develop/deploy a Typescript Smart Contract using IBM Blockchain VSCode Extension, invoke transactins/queries and render the results in a React Dashboard App"

meta_keywords: "animal tracking, queries, smart contract, IBM Blockchain, IBM blockchain platform, VSCode extension, typescript, Hyperledger Fabric, React"

last_updated: "2019-07-21"

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
    url: "https://developer.ibm.com/series/blockchain-running-enhancing-commercial-paper-smart-contract/"
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

subtitle: "Deploy a Typescript Smart Contract locally, then to IBM Cloud and integrate events/ledger data with a React-based client Dashboard app"

tags:
  - "animaltracking" "typescript" "react" "government" "dashboard" "blockchain" "ibm blockchain" "ibm blockchain platform" "ibm blockchain platform vscode extension" "ibp" "ibp2" "SaaS"

title: "Use the IBM Blockchain Platform VSCode extension to deploy a Smart Contract locally, then promote to IBM Blockchain SaaS, and integrate events and ledger data with a sample React-based client dashboard app"

type: tutorial

---

**Figure 1. "Overview of React App Dashboard interacting with the local Blockchain network" **
![AnimalTracking: Integrate summary, query and events from the blockchain](images/final-results.gif)


## Introduction

This tutorial aims to show how to integrate data and events from a blockchain ledger, into a client-side React Dashboard app. The tutorial provides the end-to-end steps to render the data and events, into a fictitious animal co-operative dashboard app, providing summary charts, tables and then query and event data sourced from a local Hyperledger Fabric runtime blockchain. We will use the IBM Blockchain Platform VS Code developer extension to start a Fabric, and invoke some sample transactions to create data. We'll also start an Event Listener, still within the extension and finally, launch the application, which will automatically render the new data in tables within the dashboard application.

**Figure 1. "CONGA Co-op" -- overview of Animal Tracking sample network and client interaction**

![Transaction flow](images/flow-overview.png)

The tutorial uses an 'intermediate' model-based Typescript contract and is aimed at Developers who wish to understand how to integrate blockchain data into a sample application, in this case a React-based application based on (Tabler UI React-based Dashboard)[https://github.com/tabler/tabler-react]. Take time to see what's going on - you don't necessarily have to understand Typescript, Javascript or React in great detail to understand this !  The lifecycle of typical transactions contained in the animal tracking contract is shown below.

![Typical contract transaction lifecycle](images/animaltracking.png)

We'll also be using the IBM Blockchain Platform VSCode Extension - and the new Fabric programming model and SDK features under the covers - to complete these tasks. In particular, you will use Query and Event Typescript application clients - in addition to the IBP VS Code extension, to perform the required actions in this tutorial.


## Background
This sample Typescript smart contract and accompanying React-based dashboard for the basis for showing end-to-end integration, from user interacting with an application client, to adding transactions, updating a blockchain ledger and querying the ledger to render results (whether aggregated summaries, filtered queries or listening for events) in a local application. The use case is an Animal Co-operative dashboard, which required the truth to be sourced from the ledger.

## Pre-requisites

1. You will need to have the following installed in order to use the extension:

* (Node v8.x or greater and npm v5.x or greater)[https://nodejs.org/en/download/]
* (Yeoman (yo) v2.x)[https://yeoman.io/]
* (Docker version v17.06.2-ce or greater)[https://www.docker.com/get-started]
* (Docker Compose v1.14.0 or greater)[https://docs.docker.com/compose/install/]
* VS Code — see the (marketplace)[https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform] for the minimum version to install
* Yarn - install yarn as follows as a privileged user:

     * `npm install -g yarn`

2. For Part 2 of this tutorial - deploying and integrating the dashboard App and smart contract to IBM Blockchain Platform SaaS - you will need to have an (IBM Blockchain Platform blockchain network)[https://cloud.ibm.com/catalog/services/blockchain] installed and running.

## Preparatory Steps

1. From a Terminal window, create a project directory (as a non-root user on Linux) called `dash`   eg assume `$HOME/dash` is the starting point 

`cd $HOME/dash` to enter that directory

2. Clone the `tabler-react` Github repository:

`git clone https://github.com/tabler/tabler-react.git`

3. Clone the `animaltracking` Github repository:

`git clone https://github.com/mahoney1/animaltracking.git`

4. IF you've previously deployed the `animal-tracking` smart contract, would suggest to perform a `teardown` in the IBM Blockchain Platform VS Code extension (click on the icon) in VS Code - then select 'FABRIC OPS', click in the '...'  select 'TearDown Runtime Fabric' and confirm you want to tear down. After doing a teardown, start a new Fabric, again from 'FABRIC OPS' .....click on 'Start New Fabric' and ensure that you have a running, functional Fabric, and with the Nodes started, in the left sidepanel.

5. In VSCode, connect to your local Fabric Gateway under the 'Fabric Gateways' sidepanel, and use the`admin` identity to connect.

6. In VSCode Explorer, choose File > Open Folder, and navigate to the `animaltracking` folder in your cloned repo - then select the `contracts` folder, eg. navigating to the `$HOME/animaltracking/contract` directory. The `contract` folder must be your top-level project folder in VSCode before proceeding

7. Click on the IBM Blockchain Platform icon and from '...' ellipses on the 'Smart Contract Packages' panel, choose to 'Package a Smart Contract' - choose `animaltracking@0.0.1`. You should get confirmation the package was successfully created.

8. Next, under 'Local Fabric Ops' choose to 'Install' the package onto the local peer - await a successful install message in VS Code.

9. Next, instantiate the `animaltracking` Smart Contract by choosing 'Instantiate'  and when prompted, select `animaltracking@0.0.1` as the contract to instantiate. 

    - When prompted to provide a function, supply the text:
      `org.example.animaltracking:instantiate` 
      and hit ENTER.
    - Hit ENTER to accept the remaining defaults for the remaining parameters when prompted. 
    
In approx. one minute or less, you should get confirmation the contract was successfully instantiated and you should see the instantiated contract called `animaltracking@xxx`, under the 'Fabric Local Ops' pane.

Successful completion of these pre-reqs, is the basis from which this tutorial can now  proceed. 

## Estimated time

After the prerequisites are completed, this should take approximately *45-60 minutes* to complete.

## Scenario

Jane Pearson has been at CONGA Co-op for 10 years now, and of late, she has taken on a very special role: she is responsible for keeping a handle on the SHEEPGOAT numbers, as they are a rare species (super-evolutionary in fact: they don't need any vaccines, tetanus jabs, immune to diseases, that affect their cousins!). Jane needs key stats about SHEEPGOATS at her fingertips. Part of her remit is to monitor events affecting SHEEPGOATS in the co-op region, be they 'green' events like new registrations, or more critical ones such as quarantined SHEEPGOATs and needs to know they are imminently being inspected by a Farm Vet.

Jane relies heavily on her dashboard app, in particular the events, from which she can instigate actions.

OK, lets get started !

## Steps

### Step 1. Perform an NPM install in the Typescript Client directory

We need to install dependencies for our client applications - to do this:

1. Change directory to the client:

`cd $HOME/dash/animaltracking/client`

2. Do an NPM install of dependencies:

`npm install`

### Step 2. Install packages for React Dashboard using Yarn

1. Change directory to the cloned `tabler-react` directory and perform an install using the yarn package manager:

`cd $HOME/dash/tabler-react`

`yarn install`

2. Copy the React dashboard customisations for this tutorial, into the `example` subdirectory in `tabler-react`, as follows:


`cd example/src`

`cp HomePage.react.js HomePage.react.js.bak`
`cp SiteWrapper.react.js SiteWrapper.react.js.bak`

(The above copies are merely so that you can do a `diff` to gauge some of the changes made, to implement the animal tracking dashboard. Check out in particular, the `json` objects, that represent JSOn coming from the blockchain, via queries and events.

3. Now copy in the customisations from the `animaltracking` cloned repo as follows **into the `example/src` subdirectory**:

`cp $HOME/dash/animaltracking/react/* .`

### Step 3. Start the React Dashboard app

1. Start the React dashboard app - go up one level, to the `example` directory:

`cd ..`

`yarn start`

2. You should get a browser launched with the Dashboard app active (when writing this tutorial, it launched Firefox on Ubuntu)
 
### Step 4. In IBP VS Code extension, pre-populate the ledger with demo data

1.  In IBP in the VSCode extension, expand the `animaltracking@xxx` contract under 'Fabric Gateways'. You will see a list of transactions. Scroll down to the end and you will find a transaction called `setupdemo`. We'll use this to pre-populate the ledger with some sample data.

2. Right-click on `setupdemo` and select `Submit Transaction` - accept all the default prompts for parameters etc - we won't need to supply any parameters.

### Step 5. In a VS Code terminal window, start the Contract Event Listener

1. Still in VS Code, click on the `Terminal` tab at the bottom - press ENTER if prompted to `hit any key to close`. You should now have a command prompt - change directory to the animaltracking `client` directory (from the `contract` subdirectory):

`cd ../client/lib`

2. Start the local Event Listener by running the bash script `listenLocal.sh`:

`./listenLocal.sh`

You should get messages ("Getting Listener" etc) that its started and you should also see an event from the earlier `setupdemo` transaction (during setup, an event for a SHEEPGOAT registration was emitted).

### Step 6. Invoke Transactions in sequence to register further events

1. Go back to the list of `animaltracking` transactions under 'Fabric Gateways' and right-click on `register` ... `Submit Transaction`.  When prompted, paste the following parameter list - including the double-quotes "" : in between the two square brackets `[ ]` in the VS Code prompt: 

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

(Note: generally, these events would picked up by an application using either Websockets (more efficient) or AJAX (less efficient) calls from the React or node application using React to render the UI. But for the purposes of this tutorial, we will write the events (emitted by the contract) to a simple JSON file: it is this that the React Dashboard app picks up, to dynamically present the ledger data in the application.

8. We also want to query all SHEEPGOAT registrations from the ledger, and present this in the dashboard. To do this, we need to run the QueryClient script as follows:

`cd $HOME/dash/animaltracking/client/lib`

`node query.js`

The script performs a number of queries, some of which are fulfilled given the ledger state at this point - however, the data we're interested in, will be the last query performed - and the results are written to a file called `registrations.json` 

9. Now copy these to our Dashboard data location:

`cp registration.json $HOME/dash/tabler-react/example/src/data`
`cp events.json $HOME/dash/tabler-react/example/src/data`

### Step 7. Check the React Dashboard App for new Query and Events

Many of the charts/area diagrams contain summarised data can can be derived from the ledger, whether aggregated, calculated or indeed - in the case of temperatures trends - data that is aggregated from IoT temperatures over a period of time - this is just an example.

The data we're interested is 'new registrations' and separately, a 'Blockchain Events' table.

1. Switch to your browser launched earlier - all the data/records we're interested in contains a grey/blue 'lock' icon

2. Check that you have a list of new SHEEPGOAT registrations - these are the list of registrations extracted by the Query client earlier.

3. Scroll down to 'Recent Blockchain Events' - these represent our 4 blockchain events emitted by the `animaltracking` Contract instantiated on channel `mychannel` and which were picked up by our Contract Listener earlier.

Well done! You've now completed this tutorial.

## Conclusion

This tutorial aimed to show a simple use case, of combining the new features of:

  - the IBM Blockchain VS Code extension to manage the whole development inside VS Code - a powerful and seamless IDE for the developer.
  - Fabric 1.4's new programming model (eg, application client SDK, gateways, wallets, event SDK enhancements, and of course the new Fabric Contract changes)
  - Typescript Contract and Client applications (Event Listeners, Query Clients etc)
  - Tabler React (the Github project at https://github.com/tabler/tabler-react for a React-based Dashboard app
  - Adding customisations, suited to the use case / Ledger data being extracted.
    
Thanks for trying it out! If you have any issues, please raise an issue on my Github project (click 'Issues') with full details of the problem you're facing - thanks!)

As a last step, it is good practice to close out your current folders in VS Code, in preparation for your next tutorial or project.
