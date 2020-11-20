---
layout: default
title: Test network
parent: Networks
nav_order: 2
---

The [Fabric test network](https://hyperledger-fabric.readthedocs.io/en/v2.2.1/test_network.html) no longer deploys FabCar chaincode by default however there are [instructions for deploying a smart contract on the test network](https://hyperledger-fabric.readthedocs.io/en/v2.2.1/deploy_chaincode.html) which are summarised here.

## Getting setup

Make sure you have [everything required for running the test network](https://hyperledger-fabric.readthedocs.io/en/v2.2.1/test_network.html#before-you-begin).

## Creation

### Smart Contract

In your working directory, clone the fabcar chaincode repo you want to use. For example,

```bash
git clone https://github.com/fabcar-samples/fabcar-chaincode-go.git
```

Ensure it can be built correctly!

### Start test network

Run the following command in the _fabric-samples/test-network_ directory to create the test network with a channel named `mychannel`

```bash
./network.sh up createChannel
```

## Contract Deploy

The chaincode needs to be installed and approved on every peer that will endorse a transaction. In this example the peers are,

- `peer0.org1.example.com`
- `peer0.org2.example.com`

After both organizations have approved the chaincode definition, one organization can commit the chaincode definition to the channel.

To operate the `peer` CLI as the Org1 admin user, you will need to set the following environment variables:

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

To operate the `peer` CLI as the Org2 admin user, you will need to set the following environment variables:

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

You can open a terminal window for each user, or reset the variables to switch between users.

### Package Chaincode

Package the fabcar chaincode you cloned earlier. E.g. for Golang,

```bash
peer lifecycle chaincode package fabcarcc.tgz --path ./fabcar-chaincode-go --lang golang --label fabcar
```

You only need to do this once.

### Install Chaincode

Install the chaincode package on the Org1 and Org2 peers, using the appropriate environment for each organization.

```bash
peer lifecycle chaincode install fabcarcc.tgz
```

### Approve Chaincode

Approve the chaincode package on the Org1 and Org2 peers, using the environment variables shown above.

To approve chaincode you need the package ID of the installed chaincode. The package ID is a combination of the chaincode label and a hash of the chaincode binaries, and is shown when installing.
The package ID can also be found using the following command.

```bash
peer lifecycle chaincode queryinstalled
```

Set an environment variable for the package ID. For example,

```bash
export CC_PACKAGE_ID=fabcar:39e56729bf6d88a08b40a76fc398eeadd2ffc2110e45b8e59576eed0b8bd4932
```

Now approve the chaincode package for Org1 and Org2, using the appropriate environment for each organization.

```bash
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name fabcar --version 1 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

### Commit Chaincode

After the chaincode definition has been approved, one organization can commit the chaincode definition to the channel. You can use the following command to check whether the chaincode is ready to commit.

```bash
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json
```

If the chaincode is ready to commit, run the following command.

```bash
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name fabcar --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```

You can confirm that the chaincode definition has been committed to the channel using the following command.

```bash
peer lifecycle chaincode querycommitted --channelID mychannel --name fabcar --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

## Run a transaction!

Create the usual cars...

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"initLedger","Args":[]}'
```

Get back all cars...

```bash
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```
