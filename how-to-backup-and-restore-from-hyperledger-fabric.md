

## How to backup and restore data from Hyperledger fabric

here we take test-network (fabric-samples v2.2.2)as an example.

### start test-network

reference this https://hyperledger-fabric.readthedocs.io/en/release-2.2/test_network.html 

![image.png](http://lifegoeson.cn:8888/images/2021/03/31/image.png)

### backup from hyperledger fabric

we need backup these four items

- crypto materials(organizations directory) 
- genesisi.block(under the directory system-genesis-block )
- peer backup files(obtain from peer container,under the path /var/hyperledger/production)
- orderer backup files(obtain from peer container,under the path /var/hyperledger/production)

create backup directory

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
mkdir -p backup/data/
```

backup crypto materials

```bash
cp -r organization backup/
```

backup genesis.block

```bash
cp -r system-genesis-block backup/
```

backup peer0.org1.example.com data

```bash
docker cp peer0.org1.example.com:/var/hyperledger/production backup/data/peer0.org1.example.com
```

backup peer0.org2.example.com 

```bash
docker cp peer0.org2.example.com:/var/hyperledger/production backup/data/peer0.org2.example.com
```

backup orderer.example.com 

```bash
docker cp orderer.example.com:/var/hyperledger/production/orderer backup/data/orderer.example.com
```

### stop network

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
./network.sh down
```

### copy backup files

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
cp -r backup/system-genesis-block/* system-genesis-block/
cp -r backup/organizations/* organizations/
cp -r backup/data/* docker
```

### configure docker-compose-test-net.yaml

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
vim docker/docker-compose-test-net.yaml
```

we need to modify volume mount path,It depends on the location of the files you backed up earlier,if 

order.example.com in the same directory as docker-compose-test-net.yaml,then you should configure volume mount path as ```./orderer.example.com:/var/hyperledger/production```,if orderer.example.com in the upper directory of docker-compose-test-net.yaml,then you should configure volume mount path as ```../orderer.example.com:/var/hyperledger/production```

![image73fe478cba0ee661.png](http://lifegoeson.cn:8888/images/2021/03/31/image73fe478cba0ee661.png)

<center>order.example.com in the same directory as docker-compose-test-net.yaml</center>

configure orderer.example.com volume mount path

```bash
##before
orderer.example.com:/var/hyperledger/production 
##after
./orderer.example.com:/var/hyperledger/production
```

![image6d8ed6a7f40bb6c5.png](http://lifegoeson.cn:8888/images/2021/03/31/image6d8ed6a7f40bb6c5.png)

configure peer0.org1.example.com volume mount path

```bash
##before
peer0.org1.example.com:/var/hyperledger/production 
##after
./peer0.org1.example.com:/var/hyperledger/production
```

![imagebdf08eadb0c70d14.png](http://lifegoeson.cn:8888/images/2021/03/31/imagebdf08eadb0c70d14.png)

configure peer0.org2.example.com volume mount path

```bash
##before
peer0.org2.example.com:/var/hyperledger/production 
##after
./peer0.org2.example.com:/var/hyperledger/production
```

![image58fbc9a5be9bf1e8.png](http://lifegoeson.cn:8888/images/2021/03/31/image58fbc9a5be9bf1e8.png)

### restore netowrk

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
docker-compose -f docker/docker-compose-test-net.yaml up -d
```

check peer0.org1.example.com logs

```bash
docker logs -f peer0.org1.example.com
```

there are lots of warn in the logs ,but dont worry ,these are normal

![image1bd6829fd6cbd9c1.png](http://lifegoeson.cn:8888/images/2021/03/31/image1bd6829fd6cbd9c1.png)

### check network 

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-samples/test-network
```

 add binaries to your CLI Path 

```bash
export PATH=${PWD}/../bin:$PATH
```

set the ```FABRIC_CFG_PATH``` to point to the ```core.yaml``` file

```bash
export FABRIC_CFG_PATH=$PWD/../config/

```

 set the environment variables 

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

```

invoke

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'

```

query

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'

```

