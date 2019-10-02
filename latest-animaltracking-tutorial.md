---
# Related publishing issue: https://github.ibm.com/IBMCode/Code-Tutorials/issues/2093

abstract: "Use the IBM Blockchain Platform VS Code extension to deploy a smart contract locally, then promote it to IBM Blockchain in IBM Cloud and integrate blockchain events and ledger data with a sample React-based client dashboard app."

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

# collection:		# Required=false 

completed_date: "2019-09-30"

components:
  - "hyperledger-fabric"
  - "hyperledger"

draft: true

excerpt: "Develop, integrate, and deploy your smart contract, and integrate ledger history and events into a React-based app."

last_updated: "2019-09-27"

meta_description: "Learn how to develop and deploy a TypeScript smart contract using the IBM Blockchain VS Code Extension, invoke transactions/queries, and render the results in a React dashboard app."

meta_keywords: "blockchain, typescript, dashboard, react, application"

meta_title: "Integrate a TypeScript smart contract with a React-based dashboard app"

primary_tag: "blockchain"

pta:
 - "emerging technology and industry"

pwg:
  - "blockchain"

# related_content:		# Required=false Note: zero or more related content
#   - type: announcements|articles|blogs|patterns|series|tutorials|videos
#     slug:
#   - type: announcements|articles|blogs|patterns|series|tutorials|videos
#     slug:

related_links:
  - title: "IBM Blockchain Platform docs: Build a network tutorial"
    url: "https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network"

# runtimes:

services:
  - "blockchain"

subtitle: "Develop, integrate, and deploy your smart contract, and integrate ledger history and events data into a React-based app"

tags:
  - "blockchain"

title: "Integrate a TypeScript smart contract with a React-based dashboard app"

type: tutorial

---

This hands-on tutorial shows how to integrate query data from a blockchain ledger and events emitted by a smart contract instantiated on a channel into a client-side React dashboard app. It uses the [IBM Blockchain Platform VS Code extension](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) as the developer platform to manage the smart contract and clients -- and essentially orchestrates the activity in this tutorial. The smart contract and the client apps, both written in TypeScript, make use of the features in the [new Hyperledger Fabric programming model](https://hyperledger-fabric.readthedocs.io/en/release-1.4/developapps/developing_applications.html), available since Hyperledger Fabric v1.4.

The IBM Blockchain Platform VS Code developer extension is used to interact with two different blockchain environments -- one a local Fabric, and the other  being the IBM Blockchain Platform in the cloud.

The tutorial flow takes you through deploying a TypeScript smart contract (using the new Hyperledger Fabric programming model) locally, and then promoting your contract to the cloud.

You will launch a locally installed React-based client dashboard, which will automatically render the source data (queries and emitted events) in the dashboard portal: The local Fabric environment is started in the VS Code extension; the cloud instance uses the [Build a network tutorial](https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network) in the IBM Blockchain Platform SaaS environment in IBM Cloud.

There are two sections:

1. **Development lifecycle** -- Deploy everything locally. Use the IBM Blockchain Platform VS Code extension to manage the full IDE, stand up the runtime local Fabric and drive transactions, and start an event listener. Then switch to the browser to show integration to the React client app.

2. **Promotion lifecycle** -- Package up the contract in the IBM Blockchain Platform extension and promote it to the cloud instance (the IBM Blockchain Platform SaaS environment). Once again, use the VS Code extension to drive the transaction flow, and again start a listener for emitted events from the cloud. Show your React app dashboard interacting with data from the IBM Cloud-based ledger.

This tutorial provides the end-to-end steps for standing up a React-based dashboard client that contains a fictitious "animal co-operative" dashboard app; this app provides summary charts and ledger-based query and event data (emitted by the contract) that's sourced from the blockchain ledger. (We're particularly interested in events related to SHEEPGOAT, a sheep/goat cross-breed!) You'll use a TypeScript-based animal tracking smart contract, and interact with it from TypeScript application clients, and of course the dashboard.

**Note:** All data that is rendered from the blockchain will have a "lock" icon alongside the record for viewing purposes.

##### Figure 1. Overview of React app dashboard interacting with the blockchain ledger for source data and events

The graphic below shows how the dashboard might look, populated as shown, once transactions are submitted to the ledger, querying the resultant data, and triggering events which get posted to the dashboard. 

![Overview of React app dashboard interacting with the blockchain ledger for source data and events](images/fig1.png)

This tutorial uses an intermediate-level, strongly typed, model-based TypeScript contract. It is aimed at developers who want to understand how to run this contract and then integrate the resulting blockchain data (as returned to app clients) into a locally installed sample ReactJS-based application, based on a [Tabler UI React-based dashboard](https://github.com/tabler/tabler-react). Take time to see what's going on -- you don't necessarily have to understand TypeScript, JavaScript, or React in great detail to understand this! Figure 2 illustrates the lifecycle of typical transactions contained in the animal tracking contract. For more details on the very useful Tabler React project, you can check out the [README and license information](https://github.com/tabler/tabler-react/blob/master/README.md).

##### Figure 2. Lifecycle of typical transactions in the animal tracking contract

![Lifecycle of typical transactions in the animal tracking contract](images/fig2.png)

As I mentioned, you'll use the IBM Blockchain Platform VS Code Extension -- and the new Fabric programming model and SDK features under the covers -- to complete these tasks. In particular, you will use the provided Query and Event TypeScript application clients, in addition to the VS Code extension, to perform the required actions in this tutorial. Figure 3 gives a general overview of the application client / SDK interaction with the smart contract that's deployed to a Fabric environment.

##### Figure 3. Overview of application client/SDK interaction with smart contract

![Overview of application client/SDK interaction with smart contract](images/fig3.png)

## Background

This use case centers around the fictitious CONGA Co-operative, and the animal cooperative's reporting needs. It uses a central dashboard to keep tabs on historical, aggregated and event-based information, and needs to source the truth from the ledger for reporting purposes.

##### Figure 4. Conga Co-operative: Overview of components involved in generating/rendering Transaction and Event data

![Background diagram](images/fig4.png)

## Scenario

Jane Pearson has been at CONGA Co-op for 10 years now, and recently she has taken on a very special role: She is responsible for keeping a handle on the SHEEPGOAT numbers (as a species their numbers need to grow). Data concerning their welfare or any incidents need to be monitored. Jane also needs key stats about SHEEPGOATs at her fingertips. While all of the data on the dashboard is important, part of her remit is to monitor information and events that affect SHEEPGOATs in the co-op region -- in particular, critical events like SHEEPGOAT quarantines or ensuring vet inspections, as well as the less important new registrations.

Jane relies heavily on her dashboard app, from which she can instigate any inline actions against events as mentioned.

## Prerequisites

1. You will need to have the following installed in order to proceed (this has been tested on a Linux virtual machine):
  
    * [Node v8.x or greater and npm v5.x or greater](https://nodejs.org/en/download/)
    * [Docker version v17.06.2-ce or greater](https://www.docker.com/get-started)
    * [Docker Compose v1.14.0 or greater](https://docs.docker.com/compose/install/)
    * [VS Code](https://code.visualstudio.com/download) editor and the IBM Blockchain VS Code extension — see the [marketplace](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) for pre-requisites
  * Install Yarn and TypeScript (if not already installed) -- see the "Preparatory steps" below to set the NPM prefix
  
2. For the second part of this tutorial -- deploying and integrating the dashboard app and smart contract to IBM Blockchain Platform SaaS -- you will need to have an [IBM Blockchain Platform blockchain network](https://cloud.ibm.com/catalog/services/blockchain) installed and running.

## Estimated time

Once the preparatory steps are completed, it should take you about 60 minutes to complete the rest of this tutorial.

## Preparatory steps

You'll need to successfully complete the following steps in order to proceed with this tutorial:

1. From a terminal window, create a project directory (as a non-root user on Linux) called `dash` -- assume `$HOME/dash` is the starting point in this example. (*Note:* If you already have TypeScript installed, you may not wish to install it locally in your environment.)
  
  ```
  cd $HOME/dash
  ```
  
2. Still in the terminal window, and your `$HOME/dash` project directory, install `typescript` and `yarn` (if they aren't already installed) via the command line as follows:
  
  ```
  npm config set prefix /home/demo/dash
  npm install -g typescript yarn
  export PATH=$PATH:$HOME/dash/bin
  ```
  
  Verify that both `tsc` and `yarn` are available in your PATH -- for example, on Ubuntu:
  
  ```
  which tsc
  which yarn
  ```
  
3. Clone the `tabler-react` GitHub repository, inside your project directory (above):
  
  ```
  git clone https://github.com/tabler/tabler-react.git
  ```
  
4. Next, clone the `animaltracking` GitHub repository:
  
  ```
  git clone https://github.com/mahoney1/animaltracking.git
  ```
  
5. If you've previously deployed the `animal-tracking` smart contract, it's a good idea to perform a `teardown` in the IBM Blockchain Platform VS Code extension (click on the icon) in VS Code -- then select **FABRIC OPS**, click in the **"..."** ellipses, select **TearDown Runtime Fabric**, and confirm that you want to tear down. After doing a teardown, start a new Fabric, again from **FABRIC OPS**; click on **Start New Fabric** and ensure that you have a running, functional Fabric inside the VS Code extension, with the Fabric nodes started, in the left sidepanel.

6. In the VS Code extension under **Fabric Gateways**, connect to your `local_fabric` sidepanel and use the `admin` identity to connect.

7. In VS Code Explorer, choose **File > Open Folder** and navigate to the `animaltracking` folder in your cloned repo; then select the `contract` folder -- for example, by navigating to the `$HOME/animaltracking/typescript/contract` directory. Depending on your VS Code version, you may be prompted to open the VS Code workspace provided in that directory. In any case, the `contract` folder must be your top-level project folder in VS Code Explorer before you can proceed.

8. In the terminal window pane, run the following command to install the dependencies for the imported TypeScript contract in the `animaltracking/typescript/contract` subdirectory (this can take up to a minute to complete):
  
  ```
  npm install
  ```

9. Click on the IBM Blockchain Platform icon and from the **'...'** ellipses on the **Smart Contract Packages** panel, choose **Package Open Project**, and choose `animaltracking-ts@0.0.1`. You should get confirmation that the package was successfully created.

10. Next, under **Local Fabric Ops** on the left, choose to **Install** the package onto the local peer, and then await a successful install message in VS Code.

11. Next, instantiate the `animaltracking-ts` smart contract by choosing **Instantiate**, and when prompted select `animaltracking-ts@0.0.1` as the contract to instantiate. When prompted to provide a function, supply the text --
  
  ```
  org.example.animaltracking:instantiate
  ```
  
  -- and hit **Enter**. Then, when prompted, select **Enter** to accept the remaining defaults for the remaining parameters.
  
  In about one minute or less, you should get confirmation that the contract was successfully instantiated and you should see the instantiated contract called `animaltracking-ts@xxx` under the **Fabric Local Ops** pane.

12. While you're still in the VS Code extension, navigate to **Nodes** under **Local Fabric Ops**, highlight `peer0.org1.example.com`, then right-click `Export Connection profile` and save it in your `$HOME` directory as filename `connection.json`. This file enables the client applications to connect to the Fabric network and in particular the configured peer.

13. From a terminal, navigate to the `animaltracking/typescript/client/` subdirectory and run the following commands:
  
  ```
  npm install
  npm run build
  ```

14. The client applications maintain configuration settings via a JSON configuration file called `clientCfg.json` located in the `animaltracking/typescript/client/cfg` directory. Navigate inside the `cfg` subdirectory from the `client` subdirectory and edit the `clientCfg.json` file. Replace the home directory with your own home directory (for example, `/home/demo`), as required. All the other settings should be good for now.

For Part 2 of this tutorial, you will need to set up an [IBM Blockchain Platform SaaS instance](https://cloud.ibm.com/catalog/services/blockchain-platform) and have completed the [Build a network tutorial](https://cloud.ibm.com/docs/services/blockchain?topic=blockchain-ibp-console-build-network) and set your IBM Blockchain Platform parameters in the same `clientCfg.json` file (more on that in Part 2).

OK, let's get started!

## Part 1. Integrate React dashboard app and blockchain ledger data with a local Fabric instance

### Step 1. Install packages for React dashboard using Yarn

1. In a new terminal window, change the directory to the cloned `tabler-react` directory and perform an install using the `yarn` package manager:
  
  ```
  cd $HOME/dash/tabler-react
  ```
  
2. Edit the `package.json` file and change the devDependency for `eslint` to be `5.6.0` explicitly -- then save the file.
  
  ```
  yarn install
  ```
  
3. Backup some existing files in `tabler-react` -- in both the `src` and `example/src` subdirectories, as follows:
  
  ```
  cd src
  cp -v Tabler.css Tabler.css.bak
  cd ../example/src
  cp -v HomePage.react.js HomePage.react.js.bak
  cp -v SiteWrapper.react.js SiteWrapper.react.js.bak
  ```
  
  **Note:** The above copies are created merely so that you can do a `diff` of the changes made, to implement the animal tracking dashboard. In particular, check out the `json` objects in `HomePage.react.js`, which represent JSON coming from the blockchain via queries and events.
  
4. Now copy in/move the customizations from the `animaltracking` cloned repo as follows. **Perform the steps in the `example/src` subdirectory** in your Tabler React directory. At this point, you can ignore the warning (in the first copy command below) about being unable to copy a subdirectory.
  
  ```
  cp -v $HOME/dash/animaltracking/react/* .
  mv -v Tabler.css ../../src
  mkdir ../public/demo/icons
  cp -v $HOME/dash/animaltracking/react/icons/* ../public/demo/icons
  ```
  
5. Create a symbolic link to where your ledger data is persisted (for this tutorial, the React dashboard app picks up data retrieved from the blockchain and persisted to a file):
  
  ```
  ln -s $HOME/dash/animaltracking/typescript/client/lib ledger
  ```
  
6. Next, you need to install the dependencies/packages in the `example` application itself (up one directory level):
  
  ```
  cd ..
  yarn install
  ```

### Step 2. Start the React dashboard app

1. Back in the terminal with the current `$HOME/dash/tabler-react/example` subdirectory earlier, start the React dashboard app -- from the current example directory:
  
  ```
  yarn start
  ```
  
2. You should get a browser launched with the dashboard app active (in my case, it launched Firefox on Ubuntu).

### Step 3. In a VS Code terminal window, start the contract Event Listener

1. Still in VS Code, click on the **Terminal** tab at the bottom and then press **Enter** if prompted to "hit any key to close." You should now have a command prompt -- change the directory to the animaltracking `client/lib` directory (from the contract subdirectory) and add execute permissions to listener scripts as shown:
  
  ```
  cd ../client/lib
  chmod +x listen*
  ```
  
2. Start the local Event Listener by running the bash script `listenLocal.sh`:
  
  ```
  ./listenLocal.sh
  ```
  
  You should see messages ("Getting listener," etc.) indicating that it has started.

### Step 4. In the VS Code extension, populate the ledger with demo data

1. In the VS Code extension, connect to the "Local Fabric" as an `admin` and expand the tree to reveal the `animaltracking-ts@xxxcontract` under "Fabric Gateways." You should see a list of transactions -- scroll down to the end and you should see a transaction called `setupdemo`. You'll use this to pre-populate the ledger with some sample data.

2. Right-click on `setupdemo` and select `Submit Transaction`. Accept all default prompts for the parameters (you don't need to supply any parameters).

3. Check the adjacent `Terminal` window in VS Code; you should see that an event has been registered for the `setupdemo` invocation.

### Step 5. Invoke transactions via the IBM Blockchain Platform extension and register further events

1. Go back to the list of `animaltracking-ts` transactions under "Fabric Gateways" and right-click on `register` ... `Submit Transaction`. When prompted, paste the following parameter list (including the double-quotes, "") *in between* the two square brackets, `[ ]`, in the VS Code prompt:
  
  ```
  "SHEEPGOAT", "000011", "24/07/2019", "BOVIS_ARIES", "FARMER.JOHN", "AVONDALE.LOC1", "ARRIVAL.F1", "IN_FIELD", "WOOL", "false"
  ```
  
2. Click on the adjacent `Terminal` window in VS Code and check that an event has been reported by the Event Listener for ID `000011`.

3. Go back to the transaction list and this time right-click on `quarantine` ... `Submit Transaction`. When prompted, paste these two parameters (each separated by double-quotes) in between the square brackets at the parameters prompt:
  
  ```
  "SHEEPGOAT", "000011"
  ```
  
4. Once again, check from the `Terminal` window that you've got an `ISOLATION` event posted.

5. Lastly, select the transaction `assigninspection` ... `Submit Transaction`. When prompted, paste the following into the square brackets:
  
  ```
  "SHEEPGOAT", "000011", "VET00007"
  ```
  
6. Once again, check the **Event Listener** pane. You should now have an INSPECT event reported.
  
  You now have four events:
  
  * an initial registration (from `setupdemo`)
  * a new SHEEPGOAT registration
  * two lifecycle events for this SHEEPGOAT
  
7. In the the `Terminal` window with the running Event Listener, use Control-C to stop it. You should see an `Events Processed` message after interruption. You should now have a set of four events, in JSON, contained in a file called `events.json`.
  
  **Note:** Generally, these events would be picked up by an application using something like Sockets.io (more efficient) or Ajax (less efficient) calls from the React or node application using React to render the UI. But for the purposes of this tutorial, the client listener writes the events (emitted by the contract) to a simple JSON file; the React dashboard app picks this up to dynamically present the ledger data in the application.
  
8. You'll also want to query all SHEEPGOAT registrations from the ledger and have the results rendered in the dashboard. To do this, you need to run the query application client script -- so from the same `client/lib` subdirectory, execute it as follows:
  
  ```
  node query.js
  ```
  
  The script performs a number of types of queries, which are fulfilled by the ledger state at this point (in particular the `setupdemo` transaction you saw earlier). However, the data we're interested in is the last `querySGRegistrations` query performed; the results are written to a file called `registrations.json`, which the dashboard will automatically pick up.
  
  **Note:** Registration ID `000011` will not show in the list in the latest registrations table because (in this sample contract) the last function performed on that registration is one of assignment to a Veterinarian (Vet).

### Step 6. Check the React dashboard app for new query and events

Many of the charts and area diagrams contain summarized data that can be derived from the ledger, whether it's aggregated or calculated. For example, in the case of temperature trends, the data is aggregated from IoT temperatures over a specific period of time.

The data we're specifically interested in however, is "new registrations" and, separately, a "Blockchain Events" table. All of the data/records that have been emitted (events) or queried (ledger data) through smart contract transaction functions, are conspicuous by the appended grey/blue "lock" icon in each row.

1. Check that you have a list of new SHEEPGOAT registrations -- this is the list of registrations that were extracted by the Query client earlier.

2. Scroll down to "Recent Blockchain Events." These represent the four blockchain events emitted by the `animaltracking-ts` contract instantiated on the channel `mychannel` and which were picked up by the Contract Listener earlier. (*Note:* The 'inline' actions on each event entry have not been coded up.)

Well done! You've now completed this first part of the tutorial.

## Part 2. Integrate React dashboard app and blockchain ledger data from an IBM Blockchain Platform SaaS instance

**Note 1:** If you wish to complete this part of the tutorial and you are not already signed up for an IBM Blockchain Platform cloud instance, you will need to do so [here](https://cloud.ibm.com/catalog/services/blockchain-platform).

**Note 2:** The IBM Blockchain Platform VS Code extension now has capability to do a 'remote deploy' of your 1.4 smart contract to the IBM Blockchain Platform enabled Fabric environment. More information on this can be found [here](https://github.com/IBM-Blockchain/blockchain-vscode-extension/releases/tag/v1.0.7). The smart contract packaged as a CDS file in step 1 below, is installed through the IBM Blockchain Platform operator console, as opposed to using the 'remote deploy' feature of the extension, at this time.

### Step 1. Export the CDS package and install/instantiate the contract in IBM Blockchain Platform

You will need to package up your TypeScript smart contract into a [CDS file](https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4noah.html#packaging) to deploy it as a chaincode package for the IBM Blockchain Platform SaaS environment. You can do this easily from the IBM Blockchain Platform VS Code extension.

1. In VS Code, click on the IBM Blockchain Platform extension icon in the sidebar, and under "Smart Contract Packages" select `animaltracking-ts@0.0.1` and right-click "Export Package." Accept the default file name `animaltracking-ts@0.0.1.cds` and export it to a temporary directory on your filesystem. You should get a "successful export" message pop-up at the bottom.

2. Log in to your IBM Blockchain Platform cloud instance and launch the IBM Blockchain Platform console. Close the welcome banner if required.

3. Click on the **Smart Contracts** tab in your IBM Blockchain Platform environment. Click on the **Install smart contract** button and when prompted, upload your CDS package from earlier and click **Install smart contract** when prompted - wait for the "success" message.
  
  ##### Figure 5. Install the smart contract
  
  ![Install the smart contract](images/fig6.png)
  
4. Next, click on the **Overflow** menu alongside the installed `animaltracking-ts` smart contract. On the side panel that opens, select the channel `channel1` to instantiate the smart contract on, then click **Next**.

5. Accept the default endorsement policy and select the Org MSP set up for your environment: `org1msp`.

6. Next, select the `Org1 peer`, which is already joined to the channel you created previously. Skip the "private data collections" prompt and click **Next** to proceed to instantiation.

7. Lastly, paste in the text `org.example.animaltracking:instantiate` when prompted to enter an initialization function. Then accept all of the defaults thereafter. Your contract should be successfully instantiated on the channel `channel` and appear in the instantiated contracts further down the same page.

### Step 2. Register an IBM Blockchain Platform identity and export the wallet for use later

1. Click on the **Org1 CA** node and click on the **Register user** button -- register a user named "ibpuser" with an enroll secret of "demo" -- "register user." This user will be the identity you use to invoke transactions, run queries, etc. from the VS Code extension later on.

2. Next, click on the action list for "ibpuser" and choose to **Enroll**. Enter the ID and secret you entered above and click **Next**.

3. Enter a display name of "ibpuser" and click on **Export identity**. This will prompt you to save the identity to your local filesystem as a JSON file -- you will use this later.

##### Figure 6. Registering users

![Registering users](images/fig7.png)

### Step 3. Export the connection profile to use in the VS Code extension as a gateway connection

1. From the **Smart Contracts** panel, scroll down to the **Instantiated smart contracts** section.

2. Click on the **Overflow** menu and select **Connect with SDK**.

3. Enter `org1msp` for the MSP choice and `Org1 CA` as the Certificate Authority of choice. Scroll down to the end and click on **Download connection profile**.

4. The exported file will be called something like `channel1_animaltracking-ts_profile.json` in your "Downloads" directory -- rename this connection JSON file locally as `connection_IBP.json` and move it to your "$HOME" directory. This directory location will be used later by the Query Client application script (which reads the setting from `clientCfg.json` in `client/cfg`) which will query the IBM Blockchain Platform SaaS channel-based ledger.
  
  ##### Figure 7. Connecting with the SDK
  
  ![Connecting with the SDK](images/fig8.png)
  
### Step 4. Set up the IBM Blockchain Platform Gateway and import the wallet to connect to the SaaS environment

1. Back in the VS Code panel in your local development environment, disconnect from any currently connected local gateways.

2. Under "Fabric Gateways," click on the "+" icon to add a new gateway. Give it a name of "IBPGW" when prompted, and when prompted, browse for the file `connection_IBP.json` in your HOME directory. Click **Select** and it should be added successfully.

3. Next, import the `ibpuser` identity which was exported as a JSON file earlier. Click on the "+" symbol under "Fabric Wallets."

4. Select **Create a new wallet and add identity** and give it a name of `ibp-wallet`. Next enter `ibpuser` in response to the "name of the identity" prompt.

5. Enter `org1msp` (or whatever your Org1 MSP is called) when prompted to enter an MSP ID, and hit **Enter**.

6. Choose **Provide a JSON file** when prompted (to choose a method for adding an identity) and browse when prompted for the file `ibpuser.json` in your HOME directory. You should see messages that it was imported successfully and it will appear on the list.

7. Now connect to the gateway under the **Fabric Gateways** panel and click on **IBPGW**; select to connect with the user "ibpuser." You will see that this user is now connected through the IBPGW gateway as highlighted in Figure 9. You are now ready to try out your integration from the React Dashboard app, all the way through to the SaaS environment. Of course, at this point you have no ledger data in the SaaS environment, so you will need to populate it and start creating some transactions.
  
  ##### Figure 8. Connecting with the gateway
  
  ![Connecting with the gateway](images/fig9.png)

### Step 5. Execute transactions and test out integration of new events and queries from the SaaS environment

**Note:** Before invoking the listener client and query client apps below, you should verify that your IBM Blockchain Platform parameters (such as MSP ID and channel name) mentioned at the start of this section have been set correctly in the `client/cfg/clientCfg.json` file. For convenience, the demo data generated in the ledger (in IBM Cloud) uses animal IDs prefixed "IBP-xx." That prefix ID can be changed as required, in the `clientCfg.json` file.

1. From the VS Code extension, expand the channel `channel` and the `animaltracking-ts@0.0.1` instantiated contract to reveal the list of available transactions -- then scroll down to the `setupdemo` transaction.
  
  ##### Figure 9. List of available transactions
  
  ![List of available transactions](images/fig10.png)

2. Click on the terminal window in VS Code and from the `animaltracking/typescript/client/lib` subdirectory, start your IBM Blockchain Platform event listener process from the command prompt as follows:
  
  ```
  ./listenIBP.sh
  ```
  
3. Right-click on `setupdemo` and click **Submit transaction**, and when prompted *supply a value of "IBP"* inside the "parameter" prompt. Hit **Enter** to accept all the remaining default prompts -- this will populate the ledger with some new registrations that are prefixed "IBP," so you can see that the data is different in your dashboard.
  
  In the **Terminal** pane in VS Code, you should get messages about an event (from `setupdemo`) being emitted from IBM Blockchain Platform in the cloud.

4. Go back to the list of `animaltracking-ts` transactions in the VS Code extension. In the instantiated contract under **Fabric Gateways**, right-click on `register` ... `Submit Transaction`. When prompted, paste the following parameter list (including the double-quotes, "") in between the two square brackets `[ ]` in the VS Code prompt:
  
  ```
  "SHEEPGOAT", "IBP-000099", "24/07/2019", "BOVIS_ARIES", "FARMER.JOHN", "AVONDALE.LOC1", "ARRIVAL.F1", "IN_FIELD", "WOOL", "false"
  ```

5. Click on the adjacent terminal in VS Code and check that you've had an event reported by the Event Listener for ID `000099`.

6. Go back to the transaction list and this time right-click on `quarantine` ... `Submit Transaction`. When prompted, paste the following two parameters (each separated by double-quotes) in between the square brackets at the parameters prompt:
  
  ```
  "SHEEPGOAT", "IBP-000099"
  ```

7. Once again, check from the terminal window that you've got an `ISOLATION` event posted.

8. Lastly, select the transaction `assigninspection` ... `Submit Transaction`. When prompted, paste the following into the square brackets:
  
  "SHEEPGOAT", "IBP-000099", "VET00007"
9. Once again, check the **Event Listener** pane. You should have an `INSPECT` event reported.
  
  You now have four events:
  
  * an initial registration (from `setupdemo`)
  * a new `SHEEPGOAT` registration
  * two lifecycle events for this `SHEEPGOAT`

10. In the terminal window with the running Event Listener, stop the Listener using Control - C inside the terminal pane. You should see an `Events Processed` message after an interruption. You should now have a set of four IBM Blockchain Platform events (with very recent timestamps) in JSON, contained in a file called `events.json`.

11. You also want to query all SHEEPGOAT registrations from the ledger and present this in the dashboard. To do this, you need to run the QueryClient script as follows:
  
  ```
  cd $HOME/dash/animaltracking/typescript/client/lib
  node query_IBP.js
  ```
  
  The script performs a number of queries, some of which are fulfilled given the SaaS ledger state at this point. The results are written to a file called `registrations.json`, which is automatically detected and rendered by the React dashboard app.

### Step 6. Check the React dashboard app for new query and events

Once again, the data we're interested in on our dashboard app are the new registrations and the "Blockchain Events" table.

1. Switch to your active React dashboard browser tab launched earlier. Once again, the data/records we're interested in contain a grey/blue "lock" icon inline, indicating it's from the blockchain network. The dashboard will momentarily refresh the new query data and events from earlier.

2. Check the refreshed list of new SHEEPGOAT registrations -- these are the registrations that are extracted by the IBM Blockchain Platform query client above and prefixed with "IBP-" for clarity.

3. Scroll down to "Recent Blockchain Events." These represent our four blockchain events (one from `setupdemo`, one from `register`, and one each from `quarantine` and `assigninspection`) that are emitted by the `animaltracking-ts` contract, instantiated on channel `channel1` in IBM Blockchain Platform SaaS (or however you configured it in `clientCfg.json`), and picked up by the IBM Blockchain Platform Contract Listener above.

Well done! You've now completed this second and final section of the tutorial.

##### Figure 10. Final Conga Co-operative dashboard view

![Final dashboard](images/fig11.png)

## Conclusion

This tutorial aimed to show a simple use case that combines the new features of:

* The IBM Blockchain VS Code extension to manage the whole development inside VS Code, a powerful and seamless IDE for developers
* Fabric 1.4's new programming model (application client SDK, gateways, wallets, event SDK enhancements, and of course the new Fabric contract changes)
* TypeScript Contract and Client applications (Event Listeners, App Clients, etc.)
* Tabler React -- see the GitHub project at https://github.com/tabler/tabler-react for a React-based dashboard app template dashboard app
* Adding customizations, suited to the use case/ledger data being extracted
* Starting local, verifying from your IBM Blockchain IDE that everything works locally, then promote the smart contract to an IBM Blockchain Platform Cloud instance, and show integration to that SaaS environment from your IBM Blockchain Platform clients and to the React dashboard app.

Thanks for trying it out! If you have any questions, please raise an issue on my GitHub project (click "Issues") at https://github.com/mahoney1/animaltracking with full details of the problem you're facing.

As a last step, it is good practice to close out your current folders in VS Code, in preparation for your next tutorial or project. 

_**Acknowledgements:** With thanks to Robert Thatcher, IBM Early Experience Programs, for his valued reviews, tutorial input and testing of the tutorial._
