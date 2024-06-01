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

#####################################################################################################################################################

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


#####################################################################################################################################################

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

#####################################################################################################################################################

Chapter 6: Dataflow introduction

Multiple data sources are supported: Google Cloud Storage, Cloud BigTable, Cloud DataStore, Pub/Sub, Big Query

Uses open source Apache Beam
Real time streaming + batch jobs

Driver program (might write using apache sdk) defines the pipeline. (Written in java or python). Pipeline -> full set of transformations that data undergoes from initial ingestion to final output.
Driver program is submitted to Pipeline runner (does execution by translating for backend execution framework). Backend system is parallel processing system
Cloud dataflow does both runner and backend.

PCollections are used in the pipelines and represent data as it is transformed within the pipeline. PCollections are made to manage transfer of data between transforms
Pipeline can be represented with DAG (directed acyclic graph) So its not possible that output of a transform becomes an input for same transform

ParDo (Parallel do) = generic parallel processing transform.

Characteristics of PCollections:
1. Can be any datatype but must all be same type
2. Does not support random access to individual elements within PCollections. Transforms are applied to each element in PCollection individually
3. Immutable. So can create new but not change
4. Bounded (finite elements) or unbounded (infinite upper limit)
5. Timestamp is with each element in PCollection. 

Event time is when a data element is created and processing time is different times at which the element is processed during its transit in the pipeline. 
Windowing: 
Basically subdivide elements of a PCollection according to timestamp.
The reason is to enable grouping/aggregation over unbounded collections

Watermarks: Beam needs to collect all the data based on the event time stamp. So it carries watermark. It is systems notion of when all the data is expected to arrive. Once it moves pass the end of the window, any further data elements that arrive, are considered to be late data. We can treat it differently. 

Triggers -> what tells beam to give out aggregated results.  

For security and permissions:
Cloud Dataflow service will use Dataflow service account
Worker instance will use Controller service account. Workers will use Compute Engine service account as the controller service account. (Can also specify user managed service account)
	Can also run metadata operations for example checking size of object

A sliding time window represents time intervals in the data stream; however, sliding time windows can overlap. This kind of windowing is useful for taking running averages of data. For example, using sliding time windows you could compute a running average of the past 60 seconds’ worth of data, updated every 30 seconds.
Triggers determine when to emit aggregated results as data arrives. For bounded data, results are emitted after all of the input has been processed. For unbounded data, results are emitted when the watermark passes the end of the window, indicating that the system believes all input data for that window has been processed.
GroupByKey is a Beam transform for processing collections of key/value pairs. It’s a parallel reduction operation, analogous to the Shuffle phase of a Map/Shuffle/Reduce-style algorithm. CoGroupByKey performs a relational join of two or more key/value PCollections that have the same key type. Combine simply combines elements, and Partition splits elements into smaller collections.
ParDo is a Beam transform for generic parallel processing. The ParDo processing paradigm is similar to the 'Map' phase of a Map/Shuffle/Reduce-style algorithm: a ParDo transform considers each element in the input PCollection, performs some processing function (your user code) on that element, and emits zero, one, or multiple elements to an output PCollection.


#####################################################################################################################################################
Chapter 7: Cloud Dataproc

Managed cluster service in GCP for running Hadoop and Spark jobs. So no hassle of running our own VM and managing storage. 
Basically, dataproc nodes are preconfigured VMs, built from google maintained image that includes Hadoop, Spark, etc. But can also create our own different image. 

According to the resources we define, Dataproc will create  master node running the YARN resource manager. + HDFS named node and worker nodes running the YARN node manager and HDFS data nodes..  Advantage: 
The billing. Cluster actions are finished in 90 seconds and pay-per-second with minimum of 1 minute. So we can quickly buildup a cluster + scale up/down. Can also do autoscaling wrt the job. + outputs can be automatically pushed to GCP, BigQuery, Bigtable  Also integrates Stackdriver logging and monitoring: to monitor the health of your Data proc cluster. 

Autoscaling: it checks memory. If available memory is more so need to scale down.
When use autoscaling, then use cloud storage otherwise have to ave enough primary workers so in scale down HDFS doesn’t get corrupted

Types of clusters:
1. single node cluster (a single VM that will run master and work the process). Can provide features of dataproc but limited to the capacity of 1 VM
2. Standard Cluster: (a master VM that runs YARN resource manager, HDFS Name Node. And two or more worker nodes with YARN node manager and HDFS data Node)
3. High availability cluster: Similar but here we have 3 master VMs 


Advanced Dataproc:
- Custom cluster properties
- Initialisation script that will be used by all nodes in our cluster. (Used for staging binaries that might be required for the job)
- Can define custom scala or java dependencies


Dataroc runs on Compute Engine VMs so we can use advanced features:
- Advanced SSDs for local or worker nodes
- Can attach GPUs




- Great choice for migrating Hadoop and Spark workloads into GCP. Benefits: 
    - Easy scaling
    - Using Google cloud storage instead of HDFS
    - Connecting to other GCP service like BigQuery and BigTable
- Ephmeral clusters
* Since preemptibles can be reclaimed at any time, preemptible workers do not store data.


#####################################################################################################################################################
Chapter 8: BigTable

Managed wide column no-SQL database.  key-value pairs where values are split into columns (diff than others with document store style of NoSQL)

Big table is sparse DB which means empty rows do not use any space in DB.
Each row has a string row_key. That is unique. Each row can have values in cells in form of array of bytes.
Trick is to save row_keys value so to store mot info in it, so its easier for querying

Column families means to have 2 columns in a single family so easy for querying

Blocks of contiguous rows are shared into tablets. These tablets are what’s managed by nodes in Bigtable cluster not the overall table itself. So workloads can be scaled easily. This tablet data is kept inside Google Colossus (internal global file system. Good for scaling cluster size).
Splitting, merging and rebalancing tablet happens automatically.

Good for:
- Marketing, financial and transactional data
- Time series data and IOT devices
- Streaming analysis
- Storage engine for batch map reduce operation and ML apps but not good for all (check alternatives)
So basically large quantities of small data. 

alternatives:
SQL + OLTP  = Cloud SQL / Spanner
Interactive SQL like queries + OLAP + cost efficiency > latency = BigQuery
Structured NoSQL documents = Firestore (can transform into wide-column format but not hierarchical like firestorm so no cost benefits if less than terabytes)
Simple Key-value pairs (not that value is different columns) = cloud memory store
Realtime applications = Firebase 


Bigtable Architecture: (see picture)
Instance: a logical container for big table clusters to share configurations. 
Two types of instance:
1. Production
2. Development
Can go from dev to prod but not vice versa. 
Can choose SSD (each node 2.5 TB) or HDD (each node 8 TB). But SSD is better because money can be wasted due to time in processing data.
Application Profiles: single or multi-cluster routing
	- if we have single row transactions then it will be hard to keep consistency in multiple clusters so we use single cluster routing
	- but if single cluster fails then we have to route it ourselves to the other one


- BigTable is a 3 dimensional table. As every cell is written with a timestamp. And if cell value is changed then new values can be written without overwriting the original values.
- Bigtable has operations by row. So in updating 2 rows if 1 doesn’t update the other will still change
- A tablet cannot be shared by more than 1 node
* If a subset of queries are performing poorly, there is likely a hot spot being caused by the row key design. Key Visualizer provides a heatmap of the load across your entire table in a way that is difficult to understand using standard debugging techniques. Increasing the size of a cluster is not a long term solution, and adding additional clusters will not affect write throughput.

