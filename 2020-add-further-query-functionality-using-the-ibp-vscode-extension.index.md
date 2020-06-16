---
# Related publishing issue: https://github.ibm.com/IBMCode/Code-Tutorials/issues/493

abstract: "Use the IBM Blockchain Platform VS Code extension to upgrade an existing commercial paper smart contract easily, and add further query capability."

authors:
  - name: "Paul O'Mahony"
    email: "mahoney@uk.ibm.com"

# collections:		# Required=false

completed_date:	"2020-06-21"

components:
  - "hyperledger-fabric"
  - "hyperledger"
  - "vscode-extension"
  
draft: true

excerpt: "Use the IBM Blockchain Platform VS Code extension to upgrade an existing commercial paper smart contract easily, and add further query capability."

last_updated:	"2020-06-21"

meta_description: "Use the IBM Blockchain Platform VS Code extension to upgrade an existing commercial paper smart contract easily, and add further query capability."

meta_keywords: "commercial paper, queries, smart contract, IBM blockchain, IBM blockchain platform, VS Code extension, Hyperledger Fabric"

meta_title: "Enhance query functionality: Add deltas history to your commercial paper sample using the IBM Blockchain VS Code extension"

primary_tag: "blockchain"

pta:
 - "emerging technology and industry"

pwg:
  - "blockchain"

related_content:
  - type: "tutorials"
    slug: "queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension"
  - type: "tutorials"
    slug: "run-commercial-paper-smart-contract-with-ibm-blockchain-vscode-extension"
  - type: "patterns"
    slug: "create-and-execute-blockchain-smart-contracts"

related_links:
  - title: "Video: Start developing with the IBM Blockchain Platform VS Code Extension"
    url: "https://youtu.be/0NkGGIUPhqk"
  - title: "Sample commercial paper smart contract"
    url: "https://github.com/hyperledger/fabric-samples"
  - title: "Hyperledger Fabric docs: Commercial paper tutorial"
    url: "https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html"

# runtimes:

services:
  - "blockchain"

subtitle: "Upgrade an existing commercial paper smart contract easily and add further query capability"

tags:
  - "finance"

title: "Upgrade a commercial paper smart contract by adding deltas history query capability"

# translators:		# Required=false The list of persons who assisted in translation.
#   - name:
#     email:

type: tutorial

---

In the tutorial [Enhance and add queries to a commercial paper smart contract with the IBM Blockchain VS Code extension](https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension), I showed you how to add rich query functionality and enhance your smart contract using the IBM Blockchain VS Code extension. That tutorial enabled you to query the history and lifecycle of a commercial paper instance and report on it. Showing that immutable history is of course important, but what if you just want to know, for example, what changes were committed for each transaction in that history? Such a use case is relevant when you're dealing with large volumes of transactions, when you're looking for patterns, etc. The example below demonstrates a smaller-scale example; nevertheless, the principles -- and indeed, the code samples provided -- apply to any ledger and use case.

Figure 1 depicts the general flow of this tutorial. Notice that the major difference (compared to the ["Enhance and add queries" tutorial](https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension/)) is that only the "deltas" are being returned by the smart contract (not the full history), in querying the history of changes over the course of a commercial paper's lifecycle (more on this later).

**Figure 1. Overview diagram**

![Overview diagram](images/delta-diagram.png)

The aim of this tutorial is to add the capability to pull just the *deltas* from the blockchain and report on them. That is:

* Add the required functionality for getting deltas only to the existing smart contract logic.
* Upgrade the existing, query-rich smart contract (from the previous tutorial) using the IBM Blockchain Platform VS Code extension.
* Try it out with an updated DigiBank client application to pull the results, as an employee of DigiBank.
* Render the deltas in an HTML table in a client browser, reporting deltas for the commercial paper lifecycle in question -- what changed after it was initially created?

You'll utilise code blocks contained in this tutorial, to add functionality to the existing contract. You'll also provide the client application changes to be able to call the new `getDeltas` query transaction. Finally, you'll display the delta history results in a simple, HTML-based table.

The goal here is to show the history of changes/deltas *only* -- so not *all* fields will show up in the HTML table; things that don't change will be blank, except for some constants like "state." (For example, two consecutive "BUY" transactions by different organizations will be the same state.)

You'll complete these tasks using the IBM Blockchain Platform VS Code extension, along with using the new Hyperledger Fabric programming model and SDK features.

## Background

There is a fantastic description of the commercial paper use case scenario in the latest [Hyperledger Fabric Developing Applications](https://hyperledger-fabric.readthedocs.io/en/master/tutorial/commercial_paper.html) docs, and the scenario depicted there makes for fascinating reading. In short, it's a way for large institutions/organizations to obtain funds and  meet short-term debt obligations -- and a chance for investors to get return on investment upon maturity.

In the previous commercial paper scenario, users transacted as participants from their respective organizations, MagnetoCorp and DigiBank. Now you'll extend the lifecycle of the commercial paper by involving another organization, Hedgematic, primarily to show more historical data in the lifecycle.

## Prerequisites

1. You need to have completed the previous tutorial in this series, [Enhance and add queries to a commercial paper smart contract with the IBM Blockchain VS Code extension](https://developer.ibm.com/tutorials/queries-commercial-paper-smart-contract-ibm-blockchain-vscode-extension).
2. Specifically, have version 0.0.2 of the commercial paper smart contract package instantiated in the IBM Blockchain Platform VS Code extension (under the "Smart Contracts" pane).
3. From the previous tutorial, you will have executed transactions, meaning you've created the transaction history or "paper trail".
4. You should have connected to your local 'Commerce' Fabric network and have `papercontract@0.0.2` shown as the instantiated papercontract version under the **Fabric Environments** pane in the IBM Blockchain Platform VS Code sidebar.
5. In VS Code Explorer, choose **File > Open Folder** and already have selected the `contracts` folder by navigating to the `$HOME/fabric-samples/commercial-paper/organization/magnetocorp` directory. This will be your top-level project folder for this tutorial.
