# Google Cloud Platform

Notes for learning Google Cloud Platform

What defines Big DATA: 
1. Volume
2. Velocity (speed at which data is being processed) Ad-hoc, batched, near-real time, real time
3. Variety (key value, tabular, streaming data, video, audio)


Data Warehouse -> data is structured or processed after transformation, ready to use, rigid structure
Data Lake -> raw/ unstructured data is stored, ready to analyse, more flexible for new types of data

OLTP Online transactional processing (modify database) -> High number of short transactional statements e.g., select, insert in SQL, fast queries OLAP Online analytical processing (query a database)-> Low volume of long-running queries, aggregated historical data from other OLTP systems

ETL extract, transform, load can take data from OLTP system to OLAP system

SQL -> structured tabular data like rows and columns e.g., MySQL, Postgres
NoSQL -> Key-value stores, Json document stores. e.g., MongoDB, Apache Casandra

INGESTION -> STORAGE -> PROCESSING -> VISUALIZATION

Cloud Storage: unstructured. Standard, nearline or cold storage. Storage event triggers by Pub/Sub
Cloud BigTable: Petabyte scale NoSQL, high throughput and scalability, (Time series, transactional, IoT data)
BigQuery: Petabyte scale data warehouse, SQL queries across large datasets, can have historical data for very little cost like cloud storage
Cloud Spanner(no AWS): proprietary database system by google but ANSI SQL compliant, horizontal scalability(like most NoSQL databases), strong consistency, for example financial transactions. (CAP theorem)It favours consistency over availability (5 9s of availablity means 5 minutes of downtime in 1 year)
The replica is not what we can connect to- but a part of cloud spanner instance. We connect through cloud spanner API. With minimum node count of 1, each replica will have 1 virtual machine. So actually the grouping of these virtual machines form a node (see picture). Each change of data requires 2/3 quorum (of replicas). We should design performant schema (so toke advantage of replicas), have compute engine in same region as Spanner. 
Locking read-write is only transaction that supports writes to database. Read only is not locking.
Cloud SQL (Amazon RDS): Builtin backups/failover, vertically scales (can add CPU or RAM) (only MySQL, PostgreSQL). Disruptive changes (updates) can take place so we can schedule them. Automatic backups can also happen which can have a small impact on performance. Point-in-time recovery (requires binary logging enabled). 
SUPER privilege is not available in cloud SQL from MySQL
Cloud Firestorm (replacing cloud datastore. Amazon DynamoDB): NoSQL, Large collections of small JSON documents, like MongoDB. Can retrieve a document by ID, get all documents in a collection, Query the documents with where clause (comparison operators or array-contains)
Cloud Memorystore (managed redistribution service in GCP/ Amazon Elasticache): In memory database/ message cache or broker, vertically scale
Basic tier: but app must manage itself cold-restart or full data flush , Standard tier: adds cross-zone replication.
While Cloud Memorystore was initially based on Redis, it now supports both Redis and Memcached.


AWS S3 buckets = GCS (cloud storage). It moves objects between storage classes as they age
Buckets can be Regional, dual-regional (roughly in the same part of world), Multi regional (uses almost all datacenter available)
4 choices regarding availability of data: Standard, Nearline, Coldline, Archive
Objects -> stored as opaque data, immutable (so cannot be changed in place, have to be rewritten). This overwrite is atomic so until the new is created, the previous is available. And objects are versioned.

GCS ADVANCED FEATURES: parallel uploads, integrity checking, whenever we request then it will deencrypt and give to us, REQUESTER PAYS
Additional costs: Operational Costs like uploading/moving data out of bucket, Network costs, Data retrieval costs from nearline cold line etc. Can cut these costs with setting Lifecycle Rules.

3 classes of operations charges:
  1. class A: expensive. For example, uploading data
2. Class B: downloading data
3. Free operations: deleting data
Network charges when we are moving the data from 1 bucket to another within a continent or another place

IAM for bulk access to buckets in a project
ACL (access control lists) for granular access 
Signed URL for temporary access to a file
Signed policy documents -> What kind of files should be allowed to upload

IAM:
Members+Roles(e.g., Instance Admin: allow members to administer all compute engine instances in a project, Pub/Sub publisher: allow members to publish messages to a pubs topic, Storage Object viewer: read only access to objects)
Service accounts can also be created for non-human tasks requiring granular authorisation. Keys can be rotated. These service accounts can be google managed or user managed
 
CLOUD SERVICE TRANSFER SERVICE: Can transfer from http,  Amazon s3, other google bucket
Can filter names/dates
Scheduled and periodic transfers
Delete objects in destination/source bucket
Teradata and Redshift support are useful for migrating customers from amazon 


CAP Theory:
Consistency, Availability, Partition tolerance 


Managed instance groups (MIGs) on Compute Engine let you operate applications like databases on multiple identical VMs
High Availability for Cloud SQL PostgreSQL instances works because: Through synchronous writes to a regional replicated disk, all writes made to the primary instance are also available to the standby instance. This configuration reduces downtime in the event of an instance or zone failure.

######################################################################################################################################################################################################

CHAPTER 4 
MAP REDUCE
For analysis in distributed systems. Map: produces key/value pairs. Key can be input from user and value can be count. Reduce: reduces the keys with same keys

Hadoop core modules:
1. Hadoop Common: contain all libraries, OS abstractions, startup scripts required for rest of Hadoop
2. HDFS: Hadoop Distributed File system: Fault-tolerant file system 
3. Hadoop YARN: Handles resource management, job scheduling, and monitoring for Hadoop jobs
4. Hadoop Mapreduce

Name node: manages access to files of clients.  file blocks can be replicated for fault tolerance. Client commits request to main node but the actual request can be fulfilled by different data nodes depending on which provides shortest path

Apache Pig: high-level language for large datasets. (Abstraction for Map-reduce). Can also write user-defined functions in python, java, groovy, ruby etc
Mainly includes:
- Merging
- Filtering
- Transformation of data

Limitation to Map reduce is the linearity. First read data then map then reduce then write to disk.
Apache Spark: general purpose cluster-computing framework
Spark needs a cluster manager and a storage system. (See picture)

Apache Kafka (originated at LinkedIn. Similar to pub/sub): distributed streaming platform. High throughput and low latency.
4 main api in Kafka:
1. Producer: allows an application to publish stream of records to a kafka topic
2. Consumer: allows an application to subscribe to 1 or more topics and process records within
3. Streams: allows an application to act as stream process itself. Useful for transforming data 
4. Connector: extend Kafka by connecting kafka to external services like relational database

Migrating Mapreduce jobs to cloud Data Flow 

Chapter 5: PUB/SUB
Pub/sub is fully supported by cloud dataflow. 

Kind of shock absorber for system. Good for exchanging messages and data in our system + can be triggers or event for certain actions and triggering some other actions

Pub/sub follows at least once delivery to every subscription. New subscriptions with same name will have no link to previous subscriptions.
Seeking can help to retain msgs even after sending acknowledgement. Can also set snapshots to go back in time.
Messages are not ordered. So can use Cloud file store or Cloud SQL and query this with time stamps for ordering but then this will have latencies

Example
Data -> pub/sub -> dataflow ne bucket me dal dia. Aur big query se us ko analysis  kr lia SQL se (https://learn.acloud.guru/course/gcp-certified-professional-data-engineer/learn/82956285-8275-1bf2-9428-7679c623aa08/a58da841-3d6a-5cf7-6943-d44f236a319c/watch)

Limitation: it can only send 10MB msg + expired of messages
Cloud Tasks: must be explicitly invoked by publisher / client as opposed to decentralised pub/sub

By default, a message that cannot be delivered within the retention time of 10 minutes to 7 days is deleted and is no longer accessible.
A push Subscription endpoint must accept an HTTPS POST with a valid SSL certificate
All orders can be sent to a single Topic. Pub/Sub guarantees at least once delivery for every Subscription, so every system that needs to be notified will require its own Subscription.
Access control can be configured at the individual resource level, for example to grant publish-only or consume-only permissions to individual topics or subscriptions.
All jobs can be sent to a single Topic. The Compute Engine VMs should share a subscription so that they can each take a job from the queue. If they each had their own subscription, they would all be performing the same jobs.
