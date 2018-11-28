# Deploying and running the PaperNet commercial paper sample

This sample is an introduction to the best practices of developing a blockchain application solution using Hyperledger Fabric. This tutorial covers downloading the sample and a Hyperledger Fabric instance, configuring the Hyperledger Fabric instance for development mode, installing and running the smart contract, then running the sample client application and using it to submit transactions to the running Hyperledger Fabric instance.

To learn more about the development of the Commercial Paper sample, including the smart contract and associated applications, see the [following documentation](../some-link-somewhere.html).

## Setting up a development environment

Before beginning, ensure you have the following installed:
- Node v8.9.x
- Python
- Docker
- A text editor of your choice.

When deploying the commercial paper sample, it's useful to have several command windows for displaying:
- logs streamed from docker containers
- smart contract logs
- issuing commands to execute the smart contract.

### Downloading the PaperNet sample

The first step is to download the sample; including the smart contracts, application, a basic Hyperledger Fabric configuration sample, and relevant scripts.

1. Download the following GitHub repository using the following command:

        $ git clone https://github.com/hyperledger/fabric-samples.git

### Downloading Hyperledger Fabric 1.3.0

After downloading the sample directory above, next download an instance of Hyperledger Fabric. The commercial paper sample is written for Hyperledger Fabric 1.3.0.

1. To download an instance of Hyperledger Fabric 1.3.0, navigate to the `infrastructure` directory of the GitHub repository cloned in the preceding step.

2. Run these commands to get the Hyperledger Fabric 1.3.0 bootstrap script
        $ curl -sSO https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh
        $ chmod +x bootstrap.sh

4. Run the bootstrap.sh script to get the docker containers needed

        $ ./bootstrap.sh -s -b      

## Hyperledger Fabric infrastructure and smart contract deployment

At this point, you have the PaperNet smart contract and application code locally, and an instance of Hyperledger Fabric 1.3.0. In the `infrastructure` directory there is a Hyperledger Fabric basic network sample. By altering this configuration, we start the network in development mode, then start Hyperledger Fabric, and finally install and instantiate the PaperNet smart contract.

### Enabling development mode

Development mode requires you to run the smart contract container locally, instead of the peer creating a Docker container. By running the smart contract locally, you can access logs easily, and can bring up new smart contract versions quickly.

For simplicity, we're going to use the Hyperledger Fabric basic-network sample. The basic-network sample is a simple Hyperledger Fabric implementation, including a peer, a certificate authority, and an orderer.

The basic-network sample is included under the `infrastructure` directory in the `basic-network` directory. The `docker-composer.yaml` file must be edited to enable development mode.

1. Navigate to the `basic-network` directory.

2. Open the `docker-compose.yaml` file.

3. Using a text editor, find the following lines:

        command: peer node start
        # command: peer node start --peer-chaincodedev=true

4. Comment out, or delete, `command: peer node start` and remove the comment around `command: peer node start --peer-chaincodedev=true`. It should look as follows:

        # command: peer node start
        command: peer node start --peer-chaincodedev=true


### Starting the Hyperledger Fabric basic-network

The next step is start Hyperledger Fabric instance. We start according to the configuration specified in the basic-network sample.

1. Navigate to the `basic-network` directory.

2. To start the Hyperledger Fabric instance, run the following command:

        $ ./start.sh

3. Next, start logspout using the following command:

        $ ../monitor_docker.sh net_basic

    Logspout is a log routing tool for outputting Docker container logs.

### Starting the smart contract locally

Development mode requires that the smart contract is run locally. By running the smart contract locally, you can choose to either output debug statements, or connect a debugger like Visual Studio Code.

For the purposes of this tutorial, we will deploy the NodeJS version of the smart contract.

1. Navigate to the `contracts/javascript` directory.

2. Run the following command to install the smart contract dependencies:

        npm install

3. Next, start the smart contract in development mode by using the following command:

        $ $(npm bin)/fabric-chaincode-node start --peer.address=localhost:7052 --chaincode-id-name=papernet:0

    To better understand this command, we'll break it down here. The `--chaincode-id-name` sets the identifier for the smart contract. The rest of the command, `$(npm bin)/fabric-chaincode-node start --peer.address=localhost:7052`, is the command to start the smart contract.

### Installing and instantiating the smart contract

Issue commands in another command window to interact with the smart contract.

1. Open a command window and navigate to the `basic-network` directory.

2. Run the following command to start the docker container that will handle `cli` commands:

        $ docker-compose -f ./docker-compose.yml up -d cli

3. Install the smart contract using the following command:

        $ docker exec cli peer chaincode install -n papernet -v 0 -p /opt/gopath/src/github.com/javascript -l node

4. Instantiate the smart contract using the following command:

        $ docker exec cli peer chaincode instantiate -n papernet -v 0 -l node -c '{"Args":["org.papernet.commercialpaper:instantiate"]}' -C papernet

## Running the PaperNet client application

Now that the smart contract has been installed and instantiated, and is running locally, the next step is to start the PaperNet sample application. The sample application serves as a demonstration of a client application invoking functions from a running smart contract.

### Setting up the client application

The client application is contained in the `application` directory of the sample.

1. Navigate to the `application/javascript` directory.

2. Run the following command to install the application dependencies:

        npm install

3. Now that everything has been installed, a local **idwallet** needs to be created that contains the credentials for a specific user, known to the Fabric infrastructure. Run the following command to create a local folder containing the required credentials:

        $ node addToWallet.js

### Running the client application

The client application can now issue transactions using the identity in the **idwallet**. Looking back at the PaperNet smart contract, there are **issue**, **buy**, and **redeem** transactions that can be performed on **commercial paper** assets.

1. To run the client application, run the following command:

        $ node application.js

    The application will connect to the Hyperledger Fabric instance, issue a commercial paper, buy the commercial paper, and redeem the commercial paper.
