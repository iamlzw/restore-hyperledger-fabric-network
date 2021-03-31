# restore-network
restore network

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
git clone 

```

### restore network
```
docker-compose -f docker/docker-compose-test-net.yaml up -d
