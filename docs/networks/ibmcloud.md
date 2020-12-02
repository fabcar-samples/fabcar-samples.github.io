---
layout: default
title: IBM Cloud
parent: Networks
nav_order: 3
---

This guide will start up a 1 Organization, 1 Peer Fabric Network and start the contract. This is done using [IBM Blockchain Platform on IBM Cloud](https://cloud.ibm.com/catalog/services/blockchain).

## Getting setup

### IBM Blockchain Platform

You will need an IBP instance! **TBC**

### IBM Blockchain Platform collection for Ansible

This Ansible collection enables you to automate the building of Hyperledger Fabric networks. 

Follow the [installation guide](https://ibm-blockchain.github.io/ansible-collection/installation.html) to get started.
There are examples using the Docker image below if you do not want to, or can not, install all of the required software.

In your working directory, clone the ansible playbooks for building a fabcar network.

```bash
git clone https://github.com/fabcar-samples/fabcar-network-ibmcloud.git
```

### CDS Tool

Creating .cds files to install chaincode on IBP usually requires the Hyperledger Fabric command line tools to be installed and configured. In this case there's an unnoficial alternative which can create .cds files without any setup required.

```bash
curl -L "https://github.com/jt-nti/cds-cli/releases/download/0.4.0/cds-0.4.0-linux" -o cds && chmod u+x cds
```

## Creation 

Now we're going to create the various assets etc that we need

### Smart Contract

In your working directory, clone the fabcar chaincode repo you want to use. For example,

```bash
git clone https://github.com/fabcar-samples/fabcar-chaincode-go.git
```

Ensure it can be built correctly!

### Build Fabric Network

First setup the credentials for connecting to your IBP instance. Copy the `ibmcloud-credentials-sample.yml` file to `ibmcloud-credentials.yml` and edit the variables using you IBP service credentials.

Then issue this ansible command to run the playbook to create a sample network.

```bash
ansible-playbook fabcar-network-ibmcloud/ansible/create-fabcar-network.yml
```

Alternatively, using the ibp-ansible docker image:

```
docker run --rm -u $(id -u) -v "$(pwd)/fabcar-network-ibmcloud/ansible":/playbooks ibmcom/ibp-ansible ansible-playbook /playbooks/create-fabcar-network.yml
```

### Get the Fabric Network configuration

When applications run they need a local identity in a wallet. In this case there's a helpful script that can pull out all the information needed. 

Run this in your working directory - some sub-directories will be created. 

```bash
npm install -g @hyperledgendary/weftility
weft import -w ./_wallets/ibmcloud -m SampleOrgMSP -j ./fabcar-network-ibmcloud/ansible/SampleOrg\ Admin.json -r
```

## Contract Deploy

### Package Chaincode

Package the fabcar chaincode you cloned earlier. E.g. for Golang,

```bash
tar -czf "fabcar.tgz" --exclude=".git*" --transform s/fabcar-chaincode-go/src/ fabcar-chaincode-go
./cds --create --lang golang --name fabcar --version 1 --module github.com/hyperledger/fabric-samples/chaincode/fabcar/go fabcar.tgz > fabcar-network-ibmcloud/ansible/fabcar.cds
```

### Install and Instantiate Chaincode

Issue this ansible command to run the playbook to install and instantiate the fabcar chaincode:

```bash
ansible-playbook fabcar-network-ibmcloud/ansible/deploy-fabcar-contract.yml
```

Alternatively, using the ibp-ansible docker image:

```
docker run --rm -u $(id -u) -v "$(pwd)/fabcar-network-ibmcloud/ansible":/playbooks ibmcom/ibp-ansible ansible-playbook /playbooks/deploy-fabcar-contract.yml
```

## Run a transaction!

For this example, we'll use the [runhfsc](https://github.com/hyperledgendary/runhfsc) node.js client to run the transactions.

```bash
npm install -g @hyperledgendary/runhfsc
```

From your working directory, run this command to point the application at the client wallets and connection profile.

```bash
runhfsc --gateway ./fabcar-network-ibmcloud/ansible/SampleOrg\ Gateway.json --wallet _wallets/ibmcloud --user SampleOrg\ Admin --channel mychannel
```

This will connect and you can see the prompt with the id and channel, type in `contract fabcar`

```
[default] SampleOrg Admin@mychannel:<contractid> - $ contract fabcar
Contract set to fabcar
[default] SampleOrg Admin@mychannel:fabcar
```

Create the usual cars...

```
submit 'initLedger'
```

Get back a car...

```
evaluate 'queryCar' '["CAR0"]'
```

Get back all cars...

```
evaluate 'queryAllCars'
```

To close the node.js client...

```
quit
```

## Clean up

Finally, there is also an ansible playbook to delete the Fabric network.

```bash
ansible-playbook fabcar-network-ibmcloud/ansible/delete-fabcar-network.yml
```

Alternatively, using the ibp-ansible docker image:

```
docker run --rm -u $(id -u) -v "$(pwd)/fabcar-network-ibmcloud/ansible":/playbooks ibmcom/ibp-ansible ansible-playbook /playbooks/delete-fabcar-network.yml
```
