## NoSQL

##### What is NoSQL?

- Refers to family of databases that vary widely in style and technology

- However, they share a common trait
  
  - Non-relational
  
  - Not standard row and column type RDBMS

- Could be referred to as "Non-relational databases"

- NoSQL databases:
  
  - Provide new ways of storing  and querying data
    
    - Address several issues for modern apps
  
  - Are designed to handle a different breed of scale - "Big Data"
  
  - Typically address more specialized use cases
  
  - Simpler to develop app functionality for than RDBMS

Characteristics of NoSQL Databases:

- They all differ technically speaking, but have a few commonalities

- Most NoSQL databases:
  
  - Are built to scale horizontally
  
  - Share data more easily than RDBMS
  
  - Use a global unique key to simplify data sharding
  
  - Are more use case specific than RDBMS
  
  - Are more developer friendly than RDBMS
  
  - Allow more agile development via flexible schemas

NoSQL Database categories:

- Key-Value

- Document

- Column

- Graph

##### NoSQL Key-Value

- Key-Value NoSQL Database Architecture
  
  <img src="images/No_SQL_Big_data/NoSQL_key-value_example.jpg" title="" alt="example key-value" data-align="center">
  
  - Least complex
  
  - Represented as hashmap Ideal for basic CRUD operations
  
  - Scale well
  
  - Shard easily
  
  - Not intended for complex queries
  
  - Atomic for single key operations only
  
  - Value blobs are opaque to database
    
    - Less flexible data indexing and querying

- Suitable Use Cases
  
  - For quick basic CRUD operations on non-interconnected data
    
    - *e.g.* Storing and retrieving session information for web applications
  
  - Storing in-app user profiles and preferences
  
  - Shopping cart data for online stores

- Unsuitable Use Cases
  
  - For data that is interconnected with many-to-many relationships
    
    - Social networks
    
    - Recommendation engines
  
  - When high-level consistency is required for multi-operation transactions with multiple keys
    
    - Need a database that provides ACID transactions
  
  - When apps run queries based on value vs key
    
    - Consider using 'Document' category of NoSQL database

- Examples: redis, Memcached, ...

##### NoSQL Document

- Document-Based NoSQL Database Architecture
  
  <img src="images/No_SQL_Big_data/NoSQL_document_example.jpg" title="" alt="Document example" data-align="center">
  
  - Values are visible and can be queried
  
  - Each piece of data is considered a document
    
    - Typically JSON or XML
  
  - Each document offers a flexible schema
    
    - No two documents need to contain the same information
  
  - Content of document databases can be indexed and queried
    
    - Key and value range lookups and search
    
    - Analytical queries with MapReduce
  
  - Horizontally scalable
  
  - Allow sharding across multiple nodes
  
  - Typically only guarantee atomic operations on single documents

- Suitable Use Cases
  
  - Event logging for apps and processes - each event instance is represented by a new document
  
  - Online blogs - each user, post, comment, like, or action is represented by a document
  
  - Operational datasets and metadata for web and mobile apps - designed with Internet in mind (JSON, RESTfulAPIs, unstructured data)

- Unsuitable Use Cases
  
  - When you require ACID transactions
    
    - Document databases can't handle transactions that operate over multiple documents
    
    - Relational database would be a better choice
  
  - If your data is in an aggregate-oriented design
    
    - If data naturally falls into a normalized tabular model
    
    - Relational database would be a better choice

- Examples: MongoDB, CouchDB, ...

##### NoSQL Column

- Column-Based NoSQL Database Architecture
  
  - Spawned from Google's Bigtable
  
  - a.k.a. Bigtable clones or Columnar or Wide-Column databases
  
  - Store data in columns or groups of columns
  
  - Column 'families' are several rows, with unique keys, belonging to one or more columns
    
    - Grouped in families as often accessed together
  
  - Rows in a column family are not required to share the same columns
    
    - Can share all, a subset, or none
    
    - Columns can be added to any number of rows, or not

- Suitable use cases
  
  - Great for large amounts of sparse data
  
  - Column databases can handle being deployed across clusters of nodes
  
  - Column databases can be used for event logging and blogs
  
  - Counters are a unique use case for column databases
  
  - Columns can have a TTL parameter, making them useful for data with an expiration value

- Unsuitable use cases
  
  - For traditional ACID transactions
    
    - Reads and writes are only atomic at the row level
  
  - In early development, query patterns may change and require numerous changes to column-based databases
    
    - Can be costly and can slow down the production timeline

- Examples: Cassandra, apache HBase, ...

##### NoSQL Graph

- Graph NoSQL Database architecture
  
  - Graph databases store information in entities (or nodes), and relationships (or edges)
  
  - Graph databases are impressive when your data set resembles a graph-like data structure
  
  - Graph databases do not shard well
    
    - Traversing a graph with nodes split across multiple servers can become difficult and hurt performance
  
  - Graph databases are ACID transaction compliant
    
    - Unlike other NoSQL databases discussed

- Suitable use cases
  
  - For highly connected and related data
  
  - Social networking
  
  - Routing, spatial, and map apps
  
  - Recommendation engines

- Unsuitable use cases
  
  - When looking for advantages offered by other NoSQL database categories
  
  - When an application needs to scale horizontally
    
    - You will quickly reach the limitations associated with these types of data stores
  
  - When trying to update all or a subset of nodes with a given parameter
    
    - These types of operations can prove to be difficult and non-trivial

### Working with Distributed Data

##### ACID vs BASE

- ACID vs. BASE consistency models
  
  <img src="images/No_SQL_Big_data/ACID_BASE_consistency_model.jpg" title="" alt="ACID vs BASE consistency" data-align="center">

- ACID definition
  
  <img src="images/No_SQL_Big_data/ACID_definition.jpg" title="" alt="ACID" data-align="center">

- ACID consistency model
  
  - Used by relational databases
  
  - Ensures a performed transaction is always consistent
  
  - Used by
    
    - Financial institutions
    
    - Data warehousing
  
  - Databases that can handle many small simultaneous transactions => relational databases
  
  - Fully consistent system

- BASE definition
  
  <img src="images/No_SQL_Big_data/BASE_definition.jpg" title="" alt="BASE" data-align="center">

- BASE consistency model
  
  - NoSQL - few requirements for immediate consistency, data freshness, and accuracy
  
  - NoSQL benefits: availability, scale, and resilience
  
  - Used by:
    
    - Marketing and customer service companies
    
    - Social media apps
    
    - Worldwide available online services
  
  - Favors availability over consistency of data
  
  - NoSQL databases use BASE consistency model
  
  - Fully available system

##### Distributed Databases

- Concepts of distributed systems
  
  - Distributed database - a collection of multiple interconnected databases
  
  - Spread physically across various locations
  
  - Fragmentation and replication
  
  - BASE model

- Fragmentation
  
  - Example:
    
    - Fragmentation input data
      
      ![Fragment](images/No_SQL_Big_data/Distributed_data_fragmentation.jpg)
    
    - Fragmentation with distributed databases
      
      ![fragment](images/No_SQL_Big_data/Distributed_data_fragment_with_db.jpg)
  
  - Fragmentating your data (partitioning, sharding)
    
    - Grouping keys lexically (for example, all records starting with A or A-C)
    
    - Grouping records by key (for example, all records with key StoreId = 123)

- Replication
  
  <img src="images/No_SQL_Big_data/Distributed_data_replication.jpg" title="" alt="Replication" data-align="center">
  
  - Protection of data for node failures
  
  - Replication: all data fragments are stored redundantly in two or more sites
  
  - Increases the availability of the data
  
  - Replicated data needs to be synchronized => could lead to inconsistencies

- Advantages of distributed systems
  
  - Reliability and availability
  
  - Improved performance
  
  - Query processing time reduced
  
  - Ease of growth/scale
  
  - Continuous availability

- Distributed databases challenges
  
  - Distributed databases bring availability, fast scaling, and global reach capabilities
  
  - Challenge: Concurrency control => consistency of data
    
    - WRITES/READS to a single node per fragment of data => data is synchronized in the background
    
    - WRITE operations go to all nodes holding a fragment of data, READS to a subset of nodes per Consistency
    
    - Developer-driven consistency of data
  
  - No Transactions support (or very limited)

##### The CAP Theorem

- The CAP theorem states that a distributed system can provide only two of three desired properties: consistency, availability, and partition tolerance. 
  
  **Consistency (C)**: Every client sees the same data. Every read receives the data from the most recent write. 
  
  **Availability (A)**: Every request gets a non-error response, but the response may not contain the most recent data.
  
  **Partition Tolerance (P)**: The system continues to operate despite one or more breaks in inter-node communication caused by a network or node failure.

- Because a distributed system must be partition tolerant, the only choices are deciding between availability and consistency.
  
  ![CAP](images/No_SQL_Big_data/CAP_Theorem.jpg)

- Partition Tolerance
  
  - Partition - a lost or temporarily delayed connection between nodes
  
  - Partition tolerance - the cluster must work despite network issues
  
  - Distributed systems cannot avoid partitions and must be partition tolerant
  
  - Partition tolerance - basic feature of NoSQL

- NoSQL: CP or AP
  
  <img src="images/No_SQL_Big_data/CAP_NoSQL.jpg" title="" alt="CAP" data-align="center">
  
  - NoSQL - a choice between consistency and availability
  
  - MongoDB - consistency
  
  - Apache Cassandra - availability

##### Challenges in Migrating from RDBMS to NoSQL Database

- RDBMS and NoSQL
  
  ![RDBMS NoSQL](images/No_SQL_Big_data/RBDMS_vs_NoSQL.jpg)

- RDBMS to NoSQL: a mindset change
  
  - Data driven model to Query driven data model
    
    - RDBMS: Starts from the data integrity, relations between entities
    
    - NoSQL: Starts from your queries, not from your data. Models based on the way the application interacts with the data
  
  - Normalized to Denormalized data
    
    - NoSQL: Think how data can be structured based on your queries
    
    - RDBMS: Start from your normalized data and then build the queries
  
  - From ACID to BASE model
    
    - Availability vs. Consistency
    
    - CAP Theorem - choose between availability and consistency
    
    - Availability, performance, geographical presence, high data volumes
  
  - NoSQL systems, by design, do not support transactions and joins (except in limited cases)

#### MongoDB

##### Overview of MongoDB

- A document and a NoSQL database

- Where data is structured in non-relational way

- What are documents?
  
  - Associative arrays like JSON objects or Python dictionaries
  
  - For example, a student document:
    
    ```json
    {
        "firstName": "Mít",
        "lastName": "Linh",
        "email": "MitHam@love.t2n"
        "studentId": 19982308
    }
    ```

- What is a Collection?
  
  - A group of stored documents
  
  - For example, all student records in Students section (`collection`) and staff records in Employees section (`collection`)

- What is a Database?
  
  - Database stores `Collections`
  
  - For example, Students and Employees collections stored in a database called CampusManagementDB

- Why use MongoDB?
  
  - Model data as you read/write, not the other way
    
    - Traditional relational databases: create schema first, then create tables
    
    - To store another field, you have to alter tables
  
  - Bring structured/unstructured data
  
  - High availability

- Where to use MongoDB
  
  - MongoDB is a popular choice of database for
    
    - Large and unstructured
    
    - Complex
    
    - Flexible
    
    - Highly scalable applications
    
    - Self-managed, hybrid, or cloud hosted

- Advantages of MongoDB
  
  - Flexibility with Schema
  
  - Code-first Approach
    
    - In relational databases
      
      - Design
      
      - then Code
    
    - In MongoDB, code first!
      
      - No complex table definitions
      
      - Write as soon as you connect to DB
  
  - Evolving Schema
  
  - Unstructured data
  
  - Querying and Analytics
    
    - MongoDB querying using MQL
    
    - Has a wide range of operators
    
    - For complex analysis use aggregation pipelines
  
  - High Availability
    
    - MongoDB is natively a highly available system:
      
      - Resilience through redundancy
      
      - No system maintenance downtime
      
      - No upgrade downtime

##### Indexes

- Why we need Indexes
  
  - Indexes help quickly locate data without looking for it everywhere
  
  - For example: in British Library, find *Designing Data-Intensive Applications* by Martin Kleppmann:
    
    - Without indexes: You will need to look through all of them
    
    - With indexes: Go to Computers/Databases/M (for Martin)

- Indexes in MongoDB
  
  - MongoDB indexes are special data structures
  
  - They store the fields you are indexing
  
  - They also store the location of document

##### Replication & Sharding

- Replication
  
  - A MongoDB Replica Set is a cluster of data bearing nodes, each maintaining a copy of the same data
    
    ![Replication](images/No_SQL_Big_data/MongoDB_replication.jpg)
  
  - Benefits of Replication
    
    - Redundancy
    
    - High availability
    
    - Fault tolerance
    
    - Misconception - Replication does not equal disaster recovery

- Sharding (Horizontal Scaling)
  
  - Data is growing beyond hardware capacity:
    
    - Scale vertically (bigger, faster hardware)
    
    - Scale horizontally (partition the data)
  
  - Benefits of Sharding
    
    - Helps with increased throughput and capacity
    
    - Could be for legal requirements

#### Cassandra

## BIG DATA

#### Introduction to Big Data

##### What is Big Data

- The four V's of Big Data
  
  - Velocity:
    
    - Description:
      
      - Data that is generated fast
      
      - Process that never stops
    
    - Attributes:
      
      - Batch
      
      - Close to real time
      
      - Streaming
    
    - Drivers:
      
      - Improved connectivity and hardware
      
      - Rapid response times
  
  - Volume
    
    - Description
      
      - Scale of data
      
      - Increased amount of stored data
    
    - Attributes
      
      - Petabytes
      
      - Exa
      
      - Zetta
    
    - Drivers
      
      - Increase in data sources
      
      - Higher resolution sensors
      
      - Scalable infrastructure
  
  - Variety
    
    - Description
      
      - Data that comes from machines, people, and processes
      
      - Structured, semi-structured, and unstructured data
    
    - Attributes
      
      - Structure, complexity, and origin
    
    - Drivers
      
      - Mobile technologies
      
      - Scalable infrastructure
      
      - Resilience
      
      - Fault recovery
      
      - Efficient storage and retrieveal
  
  - Veracity
    
    - Description
      
      - Quality, origin, and conformity of facts
      
      - Accuracy of data
      
      - Data that comes from people and processes
    
    - Attributes
      
      - Consistency and compleness
      
      - Integrity
      
      - Ambiguity
    
    - Drivers
      
      - Cost and traceability
      
      - Robust ingestion
      
      - ETL mechanisms

- Parallel Processing, Scaling, and Data Parallelism
  
  - Linear vs. parallel processing
    
    ![Linear vs parallel](images/No_SQL_Big_data/linear_vs_parallel_process.jpg)
  
  - Why parallel processing is apt for Big Data
    
    - Parallel Processing advantages
      
      - Parallel processing approach can process large datasets in a fraction of time
      
      - Less memory and compute requirements needed as set of instructions are distributed to smaller execution nodes
      
      - More execution nodes can be added or removed from the processing network depending on complexity of the problem

#### Hadoop Ecosystem

##### Introduction to Hadoop

- What is Hadoop?
  
  - Set of open-source programs and procedures
  
  - Used for processing large amounts of data
  
  - Servers run applications on cluster
  
  - Handles parallel jobs or processes

- How does Hadoop work?
  
  ![Hadoop](images/No_SQL_Big_data/Hadoop_work.jpg)

- The challenges of  Hadoop
  
  - Processing transactions (random access)
  
  - When work cannot be parallelized
  
  - When there are dependencies in the data
  
  - Low latency data access
  
  - Processing lots of small files
  
  - Intensive calculations with little data

##### Intro to MapReduce

- What is MapReduce?
  
  - Programming model used in Hadoop for processing Big Data
  
  - Processing technique for distributed computing
  
  - Consists of a Map task and a Reduce task
  
  - Can be coded in many programming languages like Java, C++, Python, Ruby and R

- Map and reduce
  
  - Input file
  
  - Map: Processes data into key value pairs
  
  - Further data sorting and organizing
  
  - Reducer: Aggregates and computes a set of result and produces a final output
  
  - MapReduce keeps track of its task by creating a unique key

- How MapReduce works
  
  <img src="images/No_SQL_Big_data/MapReduce_work.jpg" title="" alt="MapReduce" data-align="center">

- Why use MapReduce?
  
  - Parallel Computing
  
  - Process data in tabular and Non tabular forms, such as videos
  
  - Support for multiple languages
  
  - Platform for analysis and data ware housing

##### Hadoop ecosystem

- The Hadoop ecosystem is made up of components that support one another
  
  ![Hadoop](images/No_SQL_Big_data/Hadoop_ecosystem.jpg)

- Ingest data
  
  - Flume
    
    - Collects, aggregates, and transfers big data
    
    - Has a simple and flexible architecture based on streaming data flows
    
    - Uses a simple extensible data model that allows for online analytic application
  
  - Sqoop
    
    - Designed to transfer data between relational database systems and Hadoop
    
    - Accesses the database to understand the the schema of the data
    
    - Generates a MapReduce application to import or export the data

- Store
  
  - HBase
    
    - A non-relational database that runs on top of HDFS
    
    - Provides real time wrangling on data
    
    - Stores data as indexes to allow for random and faster access to data
  
  - Cassandra
    
    - A scalable, NoSQL database designed to have no single point of failure

- Analyze data
  
  - Pig
    
    - Analyzes large amounts of data
    
    - Operates on the client side of a cluster
    
    - A procedural data flow language
  
  - Hive
    
    - Used for creating reports
    
    - Operates on the server side of a cluster
    
    - A declarative programming language

- Access Data
  
  - Impala
    
    - Scalable and easy to use platform for everyone
    
    - No programming skills required
  
  - Hue
    
    - Stands for Hadoop user experience
    
    - Alloes you to upload, browse, and query data
    
    - Runs Pig jobs and workflow
    
    - Provides editors for several SQL query languages like Hive and MySQL

##### HDFS

- HDFS
  
  - HDFS is the acronym for Hadoop Distributed File System
  
  - It is the storage layer of Hadoop
  
  - Splits the files into blocks, creates replicas of the blocks, and stores them on different machines
  
  - Provides access to streaming data
  
  - HDFS uses a command line interface to interact with Hadoop

- Key features
  
  ![HDFS](images/No_SQL_Big_data/HDFS_key_features.jpg)

- HDFS concepts
  
  - Blocks
    
    - Minimum amount of data that can be read or written
    
    - Provides fault tolerance
    
    - Default size is 64MB or 128 MB
      
      
      
      
      
      ![blocks](images/No_SQL_Big_data/HDFS_block.jpg)
    
    - Each file stored doesn't have to take up the configured space size
  
  - Nodes
    
    ![Nodes](images/No_SQL_Big_data/HDFS_nodes.jpg)
  
  - Rack awareness in HDFS
    
    - Choosing data node racks that are closest to each other
    
    - Improves cluster performance by reducing network traffic
    
    - Name node keeps the rack ID information
    
    - Replication can be done through rack awareness
  
  - Replication
    
    - Creating a copy of the data block
    
    - Copies are created for backup purpoess
    
    - Replication factor: Number of times the data block was copied
  
  - Read and Write Operations
    
    - HDFS allows write once read many operations
      
      ![HDFS](images/No_SQL_Big_data/HDFS_read_write.jpg)
    
    - HDFS architecture
      
      ![HDFS](images/No_SQL_Big_data/HDFS_architecture.jpg)

##### Hive

- Hive is data warehouse software within Hadoop that is designed for reading, writing and managing tabular-type datasets and data analysis:
  
  - It is scalable, fast, and easy to use
  
  - Hive Query Language (HiveQL) is inspired by SQL, making it easier for users to grasp concepts
  
  - It supports data cleansing and filtering depending on user's requirements

- Hive and traditional RDBMS compared
  
  ![Hive](images/No_SQL_Big_data/Hive_compared.jpg)

- Hive architecture
  
  ![Hive](images/No_SQL_Big_data/Hive_architecture.jpg)

- Hive concepts
  
  - Clients
    
    ![Hive](images/No_SQL_Big_data/Hive_client.jpg)
  
  - Hive Services
    
    <img src="images/No_SQL_Big_data/Hive_service.jpg" title="" alt="Hive" data-align="center">
    
    - Hive Server to enable queries
    
    - The driver receives query statements
    
    - The optimizer is used to split tasks efficiently
    
    - The executor executes tasks after the optimizer
    
    - Metastore stores the metadata information about the tables
















