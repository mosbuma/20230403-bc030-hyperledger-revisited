# BC030 hyperledger revisited workshop

## Select a VPS

https://docs.google.com/spreadsheets/d/17WPJb7Pk423dpmv3I00aZU5gCqfCDKwoMvlj8XwiAlI/edit?usp=sharing

- put your name in the "In use by" column when you have selected a VPS

## Login to vps

install a ssh/telnet client

- Windows: putty (https://www.putty.org/)
- Ubuntu/Mac: terminal

- connect to bc030@xx.xx.xx.xx / bc030rulez

## Start a fabric network (full tutorial @ https://hyperledger-fabric.readthedocs.io/en/release-2.2/deploy_chaincode.html#start-the-network)

cd ~/fabric-samples/test-network

./network.sh up

docker ps

./network.sh down

## Interact with a smart contract via the SDK (full tutorial @ https://hyperledger-fabric.readthedocs.io/en/release-2.2/write_first_app.html)

cd ~/fabric-samples/test-network

### cleanup first

./network.sh down

### create channel

./network.sh up createChannel -c mychannel -ca

./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript

### run node.js code

cd ~/fabric-samples/asset-transfer-basic/application-javascript

npm install @grpc/grpc-js@latest

npm install

node app.js

## Deploy a smart contract to a channel (full tutorial @ https://hyperledger-fabric.readthedocs.io/en/release-2.2/deploy_chaincode.html#)

cd ~/fabric-samples/test-network

### cleanup first

./network.sh down

### package the contract

cd ~/fabric-samples/asset-transfer-basic/chaincode-javascript

npm install

cd ~/fabric-samples/test-network

peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_1.0

### setup variables to operate as organization 1 & install chaincode for organization 1

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051

peer lifecycle chaincode install basic.tar.gz
peer lifecycle chaincode queryinstalled

### get package id and set shell variable

peer lifecycle chaincode queryinstalled
export CC_PACKAGE_ID=<output of above command>

## Approve chaincode definition for org 1

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

## setup variables to operate as organization 2 & install chaincode for organization 2

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

peer lifecycle chaincode install basic.tar.gz

### get package id and set shell variable

peer lifecycle chaincode queryinstalled
export CC_PACKAGE_ID=<output of above command>

## Approve a chaincode definition for org 2

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

## check commit chaincode definition readyness

peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json

## commit chaincode definition to the channel

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"

## check actual commit

peer lifecycle chaincode querycommitted --channelID mychannel --name basic --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

## invoke chaincode

peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
