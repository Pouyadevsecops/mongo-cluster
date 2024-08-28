# mongo-cluster

This document can help you to deploy mongodb replica set cluster with docker-compose.

## A: First you need to prepare you infrastructure.

1. Deploy 3 virtualmachine on your Hypervisor as a server.

2. Install docker and docker compose on your servers.

3. Add external disk as a lvm partition and XFS format to mount you database data for increasing database read and write speed.

## B: Create the docker-compose.yml with below parameters.

```bash
version: '3.9'
services:
  mongo1:
    image: mongo:6.0
    container_name: mongo1
    restart: always
    ports:
      - 27017:27017
    volumes:
      - /data/mongodb:/data/db
      - /opt/mongodb-keyfile:/opt/mongodb-keyfile:ro
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    command: mongod --replSet rs0 --bind_ip 0.0.0.0 --keyFile /opt/mongodb-keyfile --auth
    networks:
      - mongo-network
    healthcheck:
      test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 10s

networks:
  mongo-network:
    driver: bridge
```
You must put docker-compose file on your 3 servers.
/data is the xfs format partition on external disk.

## C. You need to create mongodb-keyfile on your server for authentication.

1. Generate the key with below command
```bash
openssl rand -base64 756 > mongodb-keyfile
```
2. Give the permission that file need.
```bash
chown 999:999 mongodb-keyfile
```
3. Put the file in /opt directory that have mount point in docker-compose.
```bash
mv mongodb-keyfile /opt
```
If you check the docker-compose yaml file you can see /opt/mongodb-keyfile.

## D. change /etc/hosts for cominucate server together with name.

For example in the first server you need to have below parameters in /etc/host.

```bash
192.168.0.1 mongodb1
#Mongodb cluster 
192.168.0.2 mongodb2
192.168.0.3 mongodb3
```
On seconde server

```bash
192.168.0.2 mongodb2
#Mongodb cluster 
192.168.0.1 mongodb1
192.168.0.3 mongodb3
```
On third server

```bash
192.168.0.3 mongodb3
#Mongodb cluster 
192.168.0.1 mongodb1
192.168.0.2 mongodb2
```


## E. Deploy the docker-compose and initiate the cluster.

```bash
docker-compose -f docker-compose.yml up -d
```
docker-compose.yml is the name of my docker compose file, rename it with your docker-compose file

```bash
docker exec -it <your mongodb container> mongosh -u <mongo user> -p <mongo password> --authenticationDatabase admin
```
```bash
rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id: 0, host: "mongo1:27017" },
      { _id: 1, host: "mongo2:27017" },
      { _id: 2, host: "mongo3:27017" }
    ]
  }
)
```
## F. Check the cluster status.

```bash
rs.status()
```


