# restore-network
restore network，this project is base on fabric-samples(v2.2.2) test-nework,I has started test-network, and backup it ,then you can restore the network,you need not create channel and join channel,also you need not install chaincode,you can direct query and invoke.make sure you have done this work https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html#before-you-begin ,In simple words，you must have run test-network successfully.

### stop before fabric network

```
./network.sh down
```

check if there are network running
```
docker ps -a 
```
make sure there is no peer or orderer running

### clone repo into fabric-samples
```
cd fabric-samples
git clone https://github.com/iamlzw/restore-network.git
cd restore-network
```

### restore network
```
docker-compose -f docker/docker-compose-test-net.yaml up -d
```
### check
reference this https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html#interacting-with-the-network
```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/

# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

#invoke
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'

#query
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'


