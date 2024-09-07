# MongoDB

MongoDB is a popular, open-source NoSQL database management system designed to handle large volumes of unstructured or semi-structured data. Unlike traditional relational databases, which organize data in tables with rows and columns, MongoDB stores data in flexible, JSON-like documents called BSON (Binary JSON). This allows for a more flexible and scalable approach to data storage and retrieval.

## Key Features of MongoDB:

## 1.   Document-Oriented:
MongoDB uses a document-based model to store data. Each record is stored as a document, which can contain nested data structures like arrays and other documents.
This model is more flexible than the table-based relational model, allowing for varied data structures within the same collection.

## 2.   Schema-less:
MongoDB is schema-less, meaning you don't need to define a rigid schema before storing data. Documents within the same collection can have different fields, making it easy to evolve your data model as your application changes.

## 3.   Scalability:
MongoDB supports horizontal scaling through sharding, where data is distributed across multiple servers or clusters. This makes it suitable for handling large-scale applications with high data volume.

## 4.   High Availability:
MongoDB ensures high availability through replica sets, where multiple copies of data are maintained across different servers. If one server goes down, another can take over, ensuring no data loss.

## 5.   Rich Query Language:
MongoDB provides a powerful query language with support for filtering, sorting, aggregations, and indexing. It also supports full-text search and geospatial queries.

## 6.   Indexing:
MongoDB allows you to create indexes on any field in a document, which can significantly improve the performance of search queries.

## 7.   Aggregation Framework:
MongoDB has a powerful aggregation framework that allows you to perform data processing and transformation operations on the server side, similar to SQL's GROUP BY and JOIN operations.

## 8.   Cross-Platform
MongoDB is cross-platform and can run on various operating systems, including Linux, Windows, and macOS.

## Use Cases:

* Content Management: Flexible data model makes it ideal for CMS and blogs.
* Big Data: Efficiently handles large datasets, making it suitable for data analytics.
* Real-Time Analytics: Provides real-time data insights and analytics for applications.
* IoT Applications: Handles the diverse and high-volume data generated by IoT devices.
* Mobile Applications: Often used as a backend for mobile apps, thanks to its flexibility and scalability.

For more details you can use below link:
https://www.mongodb.com/resources/products/fundamentals/clusters



## Deploying Mongodb replicaset cluster with docker-compose.

For deploying mongo cluser with docker-compose, first you need to have 3 servers with extra xfs partition.
XFS partiton is the bestpractice format partition for all databases.


## 1. Generate rsa key for cluster authentication.

 After deploy your infrastructure,you need to generate key for authentication between your databases:

 ```bash
 openssl rand -base64 756 > mongodb-keyfile
 ```
 then you must set a volume for your key on your docker-compose file and set 999 permission on mongodb-keyfile and share  it on your 3 server.

 ```bash
 sudo chown 999:999 /opt/mongodb-keyfile 
 ```
 /opt is directory that i selected for my keyfile and I volumed in my docker-compose file.

 ## 2. Create docker-compose.yml file.

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
      - mongo1_data:/data/db
      - /opt/mongodb-keyfile:/opt/mongodb-keyfile:ro
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    command: mongod --replSet rs0 --bind_ip 0.0.0.0 --keyFile /opt/mongodb-keyfile --auth
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongo mongo:27017/test --quiet 1
      interval: 10s
      timeout: 10s
      retries: 5

 ```
