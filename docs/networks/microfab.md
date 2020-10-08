---
layout: default
title: MicroFab
parent: Networks
nav_order: 1
---

This guide will start up a 1 Organization, 1 Peer Fabric Network and start the contract. This is done using [MicroFab](https://github.com/ibm-blockchain/microfab) - a single containerized fabric network that is perfect for fast prototyping and developement. It's uses the Fabric binaries directly so it is really Fabric.

## Getting setup

### Hyperledger Fabric Command Line Tools

We need Fabric Version 2 cli binaries, you may already have these so can skip this.  

If not, to get the `peer` commands (rather than the docker images or samples directory).

```bash
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh | bash -s -- 2.2.0 1.4.4 0.4.18 -s -d
```

Ensure that the commands and configuration are setup correctly.

```bash
export PATH=$PATH:~/bin
export FABRIC_CFG_PATH=~/config
```

## Creation 

Now we're going to create the various assets etc that we need

### Smart Contract

In your working directory, clone the fabcar chaincode repo you want to use. For example,

```bash
git clone https://github.com/fabcar-samples/fabcar-chaincode-go.git
```

Ensure it can be built correctly!

### Start Microfab

First setup an environment variable:

```bash
export MICROFAB_CONFIG='{
    "endorsing_organizations":[
        {
            "name": "SampleOrg"
        }
    ],
    "channels":[
        {
            "name": "mychannel",
            "endorsing_organizations":[
                "SampleOrg"
            ]
        }
    ],
    "capability_level":"V2_0"
}'
```

Then issue this docker command to run MicroFab

```bash
docker run --name microfab --rm -p 8080:8080 -e MICROFAB_CONFIG ibmcom/ibp-microfab
```

To run in detached mode add `-d` and then use `docker logs -f microfab` to get the logs.

### Get the MicroFab configuration

When applications (including the Peer commands ) run they need a local identity in a wallet and a gateway connection profile. In this case there's a helpful script that can pull out all the information needed. 

Run this in your working directory - some sub-directories will be created. 

```bash
npm install -g @hyperledgendary/weftility
curl -s http://console.127-0-0-1.nip.io:8080/ak/api/v1/components | weft microfab -w ./_wallets -p ./_gateways -m ./_msp -f
```

Then setup the required environment variables for the peer commands; these are provided by the `weft` tool.

## Contract Deploy

### Package Chaincode

Package the fabcar chaincode you cloned earlier. E.g. for Golang,

```bash
peer lifecycle chaincode package fabcarcc.tgz --path ./fabcar-chaincode-go --lang golang --label fabcar
```

### Install Chaincode

Install the chaincode package.

```bash
peer lifecycle chaincode install fabcarcc.tgz
```

The output from this command is required for the next step so set an environment variable for the package ID. For example,

```bash
export CC_PACKAGE_ID=fabcar:39e56729bf6d88a08b40a76fc398eeadd2ffc2110e45b8e59576eed0b8bd4932
```

### Approve and Commit Chaincode

Approve the chaincode, making sure the `package-id` matches the chaincode code package identifier from the install command

```bash
peer lifecycle chaincode approveformyorg -o orderer-api.127-0-0-1.nip.io:8080 --channelID mychannel --name fabcar --version 1 --sequence 1 --waitForEvent --package-id ${CC_PACKAGE_ID}
```

Commit the chaincode

```bash
peer lifecycle chaincode commit -o orderer-api.127-0-0-1.nip.io:8080 --channelID mychannel --name fabcar --version 1 --sequence 1
```

## Run a transaction!

Create the usual cars...

```bash
peer chaincode invoke -o orderer-api.127-0-0-1.nip.io:8080 --channelID mychannel -n fabcar -c '{"function":"initLedger","Args":[]}'
```

Get back a car...

```bash
peer chaincode query  -o orderer-api.127-0-0-1.nip.io:8080 --channelID mychannel -n fabcar -c '{"function":"queryCar","Args":["CAR0"]}'
```

Get back all cars...

```bash
peer chaincode query  -o orderer-api.127-0-0-1.nip.io:8080 --channelID mychannel -n fabcar -c '{"function":"queryAllCars","Args":[]}'
```
