---
layout: post
title: Notes on Designing Data-Intensive Applications
tags: [book report]
---

I'm reading ["Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems" by Martin Kleppmann](https://learning.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) as part of a technical book club at work. We'll be covering a new chapter each week for the next ~12 weeks. Here's my notes on each chapter.

## Intro: Foundations of Data Systems

- "Data-intensive" vs "compute-intensive"
  - Data intensive if "data is [application's] primary challenge" in terms of quantity, complexity, or turnover
  - Compute intensive if CPU cycles "are the bottleneck"
  - This book focuses on data intensive

## Part 1: Foundations of Data Systems

### Ch 1: Reliable, Scalable, and Maintainable Applications

- Application requrements:
  - Functional requirements: what it should do, such as allowing data to be stored, retrieved, searched, and processed
  - Nonfunctional requirements: general properties like security, reliability, compliance, scalability, compatibility, maintainability
- Goal: "reliable, scalable, and maintainable data systems"
  - Reliability:
    - System works correctly even in the face of adversity
    - Application functions as expected
    - Tolerance for mistakes/surprises (resiliancy)
      - "Fault is not the same as a failure"
      - Fault: system component deviates from its spec
        - Hardware faults are common, so you need redundancy
        - Software errors: systematic error w/i the system
        - Human errors: leading cause of outages
      - Failure: system as a whole stops providing required service to the user
      - Designer's goal: **prevent faults from turning into failures**
    - Performs well even under load (performance)
    - Prevents abuse, unauthorized access (security)
  - Scalability: systems's ability to cope with increased load (in terms of data, traffic, or complexity)
    - Scaling gets tricky when data system is stateful
    - Design for which operations will be common vs rare (load parameters)
  - Maintainability: system doesn't prevent the team who's building it from being productive
    - Operability: make life easy for your ops team
    - Simplicity: make it easy for new folks to understand the system
    - Evolvability: make it easy to make changes (extensibility, modifiability, plasticity)
- Building blocks of data intensive applications
  - Databases: store
  - Caches: remember expensive operations for faster reads
  - Search indices
  - Stream processing (sending messages between processes for async handling)
  - Batch processing
- Hard to choose tools for the above
- Hard to combine tools
- "Composite data system" could describe pretty much every codebase I've ever worked on.
- Questions for data system designer:
  - "How do you ensure that the data remains correct and complete, even when things go wrong internally?"
  - "How do you provide consistently good performance to clients, even when parts of your system are degraded?"
  - "How do you scale to handle an increase in load?"
  - "What does a good API for the service look like?"
- Ways to prevent faults:
  - Carefully thinking about assumptions and interactions in the system
    - Minimize opportunities for human error
    - Provide non-production environments where humans can explore, experiment
  - Thorough testing: unit, integration, and manual tests
  - Process isolation
  - Allowing processes to crash and restart
  - Measuring, monitoring, and analyzing system behavior in production
  - Enable quick recovery (also mentioned in Google SRE book)

#### Questions

- What is a RAID configuration? (from section about hardware faults)
- Definitions of the ops team's responsibilities seemed very broad.
  - Preserve organizational knowledge?
  - Keeping software and platforms up to date?
- What are "key load parameters" for GitHub? Repos? Forks? Contributors?
  - "Whale" organizations like Shopify
  - Subscribers to repository
  - Contributors to repository
  - Fan out for notifications (size of community)
  - Actions that kick off a bunch of other git-heavy operations
  - Actions subscribing to every event (can overload Redis or webhook consumers)
    - Helped by moving away from RPC to message queues
    - Thinking of moving to eventing system (immutable event log)
- Does GitHub have a chaos team? YES

#### Team discussion

- Logging is a forensic tool
- Tracing example: append request id to each new request in chain
- Celebrities as lever for scaling:
  - "What do you do when Beyonce joins your app?"
  - Twitter example: used to have a dedicated server for Justin Beiber


### Ch 2: Data Models and Query Languages

- "Polyglot persistence": using both relational and non-relational datastores
- If application language is object-oriented and datastore is relational, then you have to have a "translation layer" ("impedance mismatch")
- I never would have considered storing a resume in a JSON field...but this chapter says JSON representation is "quite appropriate" for self-contained documents
  - Reduces the impedenace mismatch between the application code and storage layer
  - "Lack of schema" is often cited as an advantage
  - Multiple tables means multiple queries or lots of joins, vs one query for JSON
  - One-to-many relationships from user profile to resume sections imply a tree structure in the data; JSON makes tree structure explicit
- Can't wait to argue about normalization and denormalization in Part III!
- "Hierarchical model": old model from 1970s very similar to JSON model used by document dbs today
  - Works well for one-to-many
  - Not well for many-to-many
  - Doesn't support joins
- Other models: "Relational model" and "Network model"
- "Network model"
  - Generalization of the hierarchical model
  - Record can have multiple parents
  - Used pointers ("Access path") instead of foreign keys
    - Kinda like traversing a linked list
    - Query by moving a cursor through the db, iterating over records and following access paths
    - Application code had to keep track of paths/relationships along the way
    - Tough, bc it's like navigating around an n-dimensional data space
  - Query code was complex, inflexible
- RELATIONAL MODEL
  - Easy to read and write; don't have to worry about records relationships or access paths
  - "Query optimizer" automatically decides how to execute query so developer doesn't have to (easier for developer, harder for machine)
  - Better for joins
  - Better for many-to-one and many-to-many relationships
  - "Schema-on-write": enforced on write
  - Appropriate for simple cases of many-to-many relationships
- DOCUMENT MODEL
  - Better schema flexibility
  - Better performance due to locality
    - Only applies if you need large parts of the document at the same time
    - Wasteful if you only need small part of the document
  - Appropriate if application has mostly one-to-many relationships or no relationships between records
  - Mirrors data structures used by application code (sometimes)
  - Don't enforce a schema on the data
  - "Schema-on-read": inferred on read
  - Can't make direct references to nested items
  - On update, rewrite the entire document (hard to update just one part)
  - NoSQL system accidentally reinventing SQL:
    - RethinkDB supports relational-like joins in its query language
    - MongoDB resolve document references w/ a client-side join
    - MongoDB has support for declarative query language "aggregation pipeline"
- GRAPH MODEL
  - Two kinds of objects: vertices (nodes, entities) and edges (relationships, arcs)
  - Kinda like two relational tables, one for vertices & one for edges
  - Good for evolvability
  - Good for use case where anything is potentially related to everything
  - Examples:
    - Social graphs
    - Web graph
    - Road or rail networks
  - Well known algorithms:
    - Shortest path between two points in a network/grid
    - PageRank
  - Used by Facebook
  - Property graph model
    - Neo4j, Titan, InfiniteGraph
    - Vertex has unique identifier, outgoing edges, incoming edges, collection of key-value pair properties
    - Edge has unique identifier, edge start ("tail"), edge end ("head"), label, collection of key-value pair properties
  - Triple store model
    - Datomic, AllegroGraph
    - Independent of semantic web, but often mentioned together
- Query languages
  - Declarative
    - Give the computer the what, not the how
    - More concise
    - Hides implementation details of the db engine, so easier to optimize without having to change any queries
    - Examples: SQL, CSS, XSL
    - Better for parallel execution
  - Imperative
    - Tell the computer what and how
    - Most programming languages are imperative
    - Examples: IMS, CODASYL
  - MapReduce
    - Must be pure functions (no additional db queries, no side effects)

#### Conclusions

- Choose the datastore that makes your application code the simplest
- "A hybrid of the relational and document models is a good route for databases to take in the future"

#### Research

- Multi-table index cluster tables
- Column family concept in Bigtable data model
- Distributed query execution (MapReduce, SQL)
- Possible to use JavaScript with some SQL dbs?! https://blog.heroku.com/JavaScript_in_your_postgres
- Recursive common table expressions

#### Questions

- What are examples of "specialized query operations" not well supported by relational model?
- Does anyone have examples when a relational schema has been "too restrictive"? How would a non-relational datastore be an improvement?
- Has anyone worked with a "polyglot persistence" model? What did you like? What did you not like?
- Has anyone chosen to use the document model and then discovered that was a bad choice? Why did you chose it? What didn't work for your use case?
  - Good case: http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/
- What is the Google Spanner database?
- SPARQL: the perfect triple store query language for GitHub!

#### Team discussion

- Interesting that key value stores weren't included
  - Maybe because there's not really a query language for it?
  - Maybe later in the book?
- Also didn't include time series stuff, other querying things
- Interesting: no XML databases. Never took off? Money got behind other things?
- MongoDB
  - Single threaded
  - Used to lock the entire database. Now locks just the document
- CSS as a query language
- Data logging
- What would building an event sourcing layer on top of Datomic be like?
- We're kind of unique because we built our service on top of Git, essentially, so not really abstracting anything in our data layer
- Have we ever wanted to use Elasticsearch for denormalized background search?
  - Related to stream processing, secondary indices
  - Problem of eventual consistency... have to be ok with some lag / inconsistencies
- Can we SOLVE EVERYTHING with change data capture?
- How does data residency affect Elasticsearch? (ex. countries, Enterprise)
- We've pushed MySQL really far... are we looking into alternatives, especially for data residency?


### Ch 3: Storage and Retrieval

- Optimized for transactional workloads vs analytics
- World's simplest db: key value store
- Note: "log" here means an "append-only sequence of records"
- Index speeds up reads, slows down writes
- Hash Tables
  - "Tombstone": special deletion record
  - Append-only > overwriting
    - "Appending and segment merging are sequential write operations, which are generally much faster than random writes"
    - Simpler concurrency and crash recovery
  - Used by Riak, Bitcask
- SSTables
  - "Sorted string table"
  - Like key value store, but sorted by key
  - Writes now occur in order
  - Can use mergesort algorithm to merge segnments
  - Can jump to offsets since you know the order; don't need to scan through all keys
  - Use red-black trees or AVL trees to _insert keys in any order and read them back in sorted order_ (!!)
  - Used by LevelDB and RocksDB
  - Introduced in Google's Bigtable paper
  - LSM-Tree
    - "Log Structured Merge Tree"
    - Keep a cascade of SSTables that are merged in the background
    - Used by Lucene (used by Elasticsearch and Solr)
    - Use "Bloom filter" to approximate contents of a set; can tell you if a key doesn't exist in a DB (HOW?)
    - Pros:
      - Faster for writes
      - Lower "write amplification"
      - Can be compressed better (smaller files on disk than B-trees)
    - Cons:
      - Slow when looking up keys that don't exist
      - Compaction can interfere with read/write performance
      - Need monitoring to catch degraded performance
- B-Trees
  - Most common indexing structure
  - Standard index implementation in almost all relational databases
  - Many non-relational dbs use them too
  - Key value pairs sorted by key => efficient lookups and range queries
  - Break db down into blocks or pages, read or write one page at a time
    - Mirrors underlying hardware (disks are also arranged in fixed size blocks)
  - "Branching factor": number of references to child pages
    - Amount of space required to store page refs and range boundaries
  - Write operation overwrites page on disk (unlike LSM trees which append, then compact, never modify in place)
  - Overwriting is risky, so also use a write-ahead log, which is an append-only file
    - Helps recover from crashes
    - But now you're always doing two writes: one to WAL, one to page
    - "Write amplificiation"
      - SSDs can only overwrite blocks a limited number of times before wearing out
      - Eats up disk bandwidth
  - Also requires latches (lightweight locks) for concurrency control
  - Alternatives:
    - Copy-on-write (used by LMDB db)
    - Abbreviate keys
    - Attempt to keep leaf pages in sequential order (kinda tough)
    - Additional pointers to sibling pages
  - Pros:
    - Faster for reads
    - More predictable query times
  - Cons:
    - Higher write amplification
    - Unused disk space due to fragmentation, so larger file size on disk
- "Clustered index": storing all row data w/i index
  - speeds up reads
  - adds overhead on writes
- "Nonclustered index": storing references to data w/i index
  - avoids duplicating data
- "Covering index": stores some of row data w/i index
  - speeds up reads
  - adds overhead on writes
- "Concatenated index": combines several fields into one key
  - query several columns at once
  - good for geospatial data
  - good for search filter combinations
- Online transaction processing (OLTP) vs Online analytic processing (OLAP)
  - Better to have separate dbs for each, as they're very different access patterns
  - Separate data warehouse for analytics queries

#### Questions

- How do Bloom Filters work? See https://en.wikipedia.org/wiki/Bloom_filter
- Appreciate discussion of hardware specific concerns (example: magnetic hard drives perform sequential writes faster, so better to use an LSM tree)
- Need to review last sections on column and cube storage

#### Team discussion

- Talked about enforcing efficient queries in the application code itself
  - Company had less trust in its devs' ability to write efficient queries
- Disk access is cached in memory
- Recommendation: give half the RAM to Elasticsearch, give the rest to file system cache to speed things up
- BTree: not a binary tree
- Search indices are essentially LSM trees
- Still only 4 disk hops in 256 terrabyte drive (thanks to page sizes)
- Bloom filters
  - Basically Cassandra
  - Weird data structure, kinda like a set
  - Ask "have you seen this key before?"
    - Sometimes it lies to you
    - But so fast, we don't mind
    - Only lies in one direction
      - If it says "no", you're golden
      - A "yes" is only probaby true (might wanna search for it anyway)
  - In-memory
  - Much smaller than an actual set
- [Ink and Switch Lab](https://www.inkandswitch.com/local-first.html)
  - Conflict free collaboration with shared data
  - Immutable logs you can rebuild data structures from
  - Author Martin Kleppmann is a member!
- Multi-dimensional indices!
- Geospatial databases do a lot of different things than dbs discussed in this chapter. Hard to handle > 20 indexes.
- Search is a one dimensional index
  - You can trick it into thinking it's 2D: https://en.wikipedia.org/wiki/Hilbert_curve
  - "Space filling curves"
- Lucene talk about GIS geospatial stuff
  - Overload those concepts for search

### Ch 4: Encoding and Evolution

- Forward compatibility harder than backward compatibility
- Programs typically represent data two ways:
  1. In memory
    - Objects, structs, lists, etc
    - Optimized for efficient updates by CPU usually using pointers
  2. Sequence of bytes
    - Necessary to write data to file or send over network
- Translation from memory to bytes: ENCODING aka "serialization", "marshalling"
- Bytes to memory: DECODING aka "parsing", "deserialization", "unmarshalling"
- Most languages have built in support for encoding in-memory objects into bytes (ex. Ruby: `Marshal`, Python: `pickle`) BUT don't use it
  - Can't really be reused w/ another language
  - Often don't handle data versioning
  - Often inefficient
- Textual data formats
  - JSON: subset of JavaScript
  - Ambigous number encoding (string vs float vs integer, etc)
  - JSON and XML don't support binary strings
  - Binary JSON encodings not much smaller than the JSON itself
- Binary encodings based on schemas
  - Thrift and Protocol Buffers
    - Require schema
    - Code generation tool that creates objects/classes from schema
    - Code generation:
      - Efficient in-memory structures
      - Allows type checking, IDE autocompletion
      - HOWEVER kinda pointless for dynamically-typed language like Ruby or JavaScript, because there's no compile-time type checker
    - Encoded record is the "concatenation of its encoded fields"
    - Fields identified by field tags, not field names (kinda like aliases)
      - Field tag == number
      - Easy to change field names
      - Helps make code backwards and forward compatible
    - 1) Thrift
      - Facebook
      - Two different formats: BinaryProtocol and CompactProtocol
      - Supports nested lists
    - 2) ProtoBuf
      - Google
      - Most size-efficient
      - Doesn't have a list or array datatype. Uses `repeated` marker instead
  - Avro
    - Uses schema, two schema languages
      - Avro IDL: human readable
      - Another one based on JSON: machine readable
    - More compact than Thrift or ProtoBuf
    - No field tags
    - Friendlier to dynamically-generated schemas
    - On write, uses "writer's schema" (schema version currently known to app)
    - On read, uses "reader's schema" (schema application code relies on)
    - Writer's schema and the reader's schema can be different as long as they're compatible. Avro knows how to resolve differences on read
    - Writer's schema version number included in every db record
    - Db stores list of writer's schema versions
    - Optional code generation (for statically typed languages)
    - Used by Linkedin's document db
- Forefathers of binary data encoding:
  - ASN.1
    - Schema definition language still used to encode SSL certs
    - Complex, poorly documented
  - Database drivers that decode responses from db's network protocol into in-memory data structures
- Binary encodings based on schemas > textual data formats
  - More compact
  - Schema provides documentation
  - Support backwards/forwards compatibility and type checks
- Dataflow Modes
  - Via databases
    - Write process encodes, read process decodes
    - Need to maintain forward compatibility to avoid data loss
  - Via service calls (REST, SOAP, and RPC)
    - "Microservices architecture" is same as "service-oriented architecture" (SOA)
    - Clients submit and query data from service
    - Requests expect a response as quickly as possible
    - Servers updated first, clients second, so requests need to be backwards compatible and responses need to be forward compatible
    - HTTP or web services: REST, SOAP
    - REST
      - Not a protocol
      - Design philosophy based on HTTP principles
      - Emphasizes simple data formats
      - OpenAPI: definition format, aka Swagger, used for documentation
      - Better for experimentation and debugging (thanks to `curl`)
    - SOAP
      - XML-based protocol for network requests
      - Used over HTTP, but aims to be independent from HTTP
      - Avoids most HTTP features
      - Uses "Web Services Description Language" (WSDL)
      - WSDL enables code generation, so client can access service using local classes and method calls
      - Useful for statically typed languages, less so dynamically typed langues
      - Not very human readable, so relies heavily on tools, IDEs
    - RPC (remote procedure call)
      - Created in 1970s
      - Tries to make network request look the same as calling a function or method (an abstraction known as "location transparency")
      - Fundamentally flawed (!!)
        - Because a local function call is predictable, but network request is unpredictable (speaking generally)
        - Local function call always returns something, but network request response might be lost completely
        - Hard to pass complex objects via network request, because you have to encode everything into bytes first
        - If client and service are implemented in different languages, RPC framework has to translate, which can get gnarly
      - Conclusion: don't bother trying to hide that you're making a network request (or as the book says, RPC is pointless)
  - Via async message passing
    - Somewhere in-between RPC and databases (what does this mean?)
      - Request delivered to process with low latency
      - Request isn't sent via direct network connection; goes via message broker that temporarily stores the message
    - Message brokers are super helpful
      - Acts as buffer
      - Can be used for redelivery
      - Prevents messages from being lost
      - Sits in front of service, so hides IP address, port, etc
      - 1 message can go to multiple recipients
      - Decouples sender from recipient
    - Sender normally doesn't expect to receive a reply to its messages (asynchronous, fire and forget, one-way data flow)
    - Actor model: actor has local state, communicates by sending and receiving messages

#### Questions

- Any examples of decoding as a source of security problems (ex. remote code execution)?
  - From footnotes:
    - https://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/
    - http://cwe.mitre.org/data/definitions/502.html
    - https://www.kalzumeus.com/2013/01/31/what-the-rails-security-issue-means-for-your-startup/
- Avro has some interesting design decisions:
  - Doesn't allow NULL as a default value; instead you have to use a `union type` and specify expected "not null type".
  - Also uses `union type` instead of markers like "optional" or "required"
- Is everything message passing? Ex. concept of "storing something in the database as sending a message to your future self"
- What do we know about Distributed Component Object Model (DCOM)?
  - Microsoft's tech for making network requests
  - Based on RPC

#### Team discussion

Erlang rules!

## Part 2: Distributed Data

- Why distribute?
  - Scalability
  - Fault tolerance
  - High availability
  - Low latency
- "Shared memory/disk architecture"
  - Cost grows linearly
  - Limited fault tolerance
- "Shared nothing architecture"
  - Horizontal scaling / "scaling out"
  - Network of nodes
  - Each node uses resources (CPUs, RAM, disks) independently
  - Adds complexity
  - Limits expressiveness of data models
  - Requires extra caution/consideration from developer

### Ch 5: Replication

- This chapter assumes dataset is small enough for a single machine (no partitioning, that's for next chapter)
- Increases availability
- Improves performance (read throughput)
- Reduce latency (keep data geographically close to users)
- What is a replica? A node that stores a copy of the db
- Three popular approaches:
  - Single leader
  - Multi-leader
  - Leaderless
- Leader based replication
  - "active/passive"
  - Most common solution
  - Leader(s) + followers (aka replicas, secondaries, hot standbys)
  - Writes are only accepted on the leader
  - Client can read from leader or followers
  - Used in dbs and message brokers (Kafka, RabbitMQ)
  - Synchronous replication
    - "Guaranteed" followers are up to date with leader
    - Writes are blocked waiting for "ok" from followers
    - "Semi-synchronous": impractical for all followers to be synchronous (usually just one)
  - Asynchronous replication
    - If leader fails, any writes that haven't been replicated are lost
    - Write is not guaranteed to be durable (!!)
    - Higher performance, but lower durability
  - Consistency vs consensus (several nodes agree on a value)
  - How do you add a new follower?
    - Lock db (causes downtime)
    - No downtime:
      - Copy data snapshot ot new node
      - Node requests all changes since snapshot from leader
      - Node processes the backlog, then it's caught up
  - Similar process for follower recovering from crash (catch up from changelog)
  - Failover: choose a new leader, reconfigure hierarchy
    - How do nodes decide on a new leader? (consensus)
    - Does new leader have all the writes?
    - What happens if old leader comes back up?
    - More than one node thinks it's the leader ("split brain")
    - What's the right timeout? (When do you kill the leader?)
  - Statement based replication
    - Leader logs every write ("statement")
    - Dicey because of nondeterministic functions (ex. `NOW()` has different values on different nodes), statements must be executed in same order, other side effects
  - Write Ahead Log Shipping (WAL)
    - Append-only sequence of bytes containing all writes
    - Use this same log to build replicas
    - Used in PostgreSQL and Oracle
    - Because data is described at the byte level, this method is closely tied to storage format (and db version)
    - Upgrades could require downtime
  - Logical (row-based) log replication
    - Decoupled from storage engine internals
    - "Change data capture": Contains data column data for affected row
    - Backward compatible
    - Parseable by external applications (ex. data warehouse)
  - Replication happens at db eleve, but can use triggers and stored procedures to run from application layer
    - Greater overhead
    - More prone to bugs
  - "Read-scaling architecture": distribute read requests across many followers
    - Good for workloads w/ few writes
    - Kinda have to use async replication (all nodes would be blocked otherwise)
  - Replication lag complications:
    - "Read after write" consistency
      - aka "read your writes" consistency
      - User can submit data, reload page, see what they just submitted
      - Read current user data from leader, everything else from followers
      - "cross-device read-after-write" consistency: when user accesses your service from multiple devices
    - Monotonic reads
      - User will not read older data than previously read data
      - Could always have user read from same replica, but then what if it fails?
    - Consistent prefix reads
      - Prevent "time traveling" into the future
      - Anyone reading writes will read them in the same order they happened
      - Big problem in partitioned/sharded dbs
  - Multi-leader Replication
    - aka "active/active" replication
    - More than one node can accept writes
    - Still forward writes to other nodes
    - Better perceived performance
    - Tolerates network problems better
    - Problem: resolving write conflicts
    - When does this config make sense?
      - Multi-datacenter operation
        - Replicas in several datacenters
        - Ex. have a leader in each datacenter
      - Offline operation
        - Local database is the leader, writes to other leader whenever it's online (async replication)
        - Device is its own "datacenter"
      - Collaborative editing (Google Docs)
        - Concurrently edit single document
        - Local replica, changes applied async to server
        - Need to handle write conflict resolution in real time
      - Mostly should be avoided if possible: "As multi-leader replication is a somewhat retrofitted feature in many databases, there are often subtle configuration pitfalls and surprising interactions with other database features. For example, autoincrementing keys, triggers, and integrity constraints can be problematic."
    - Write Conflict Resolution
      - Sync vs async conflict detection
      - Try to avoid it (how is this even possible??)
      - Single leader applies writes in sequential order
      - Order is less clear w/ multi-leader
      - Last Write Wins
        - Compare, pick latest, throw away the other writes
        - Popular option
        - Prone to data loss
        - "achieves...eventual convergence, but at the cost of durability"
      - Merge values (?!)
        - This doesn't make any sense to me
      - Record conflict in its own data structure that preserves all the data
      - Write application code that resolves conflict somehow
        - Resolve on write
        - Resolve on read
        - CRDTs
        - MPDSs (mergeable persistent data structures)
        - Operational transformation
    - Replication Topologies
      - Communication paths to propogate writes to all nodes
      - Circular
      - Star
      - All to all
- Leaderless replication
  - Used by Amazon's Dynamo, Riak, Cassandra, Voldemort
  - Reads and writes sent to several replicas
    - Use version numbers to determine which value is newest
    - "Quorum" reads and writes
    - Quorums are not necessarily majorities
  - Sometimes use coordinator node to send writes to replicas
    - Coordinator doesn't enforce ordering of writes (?!)
  - Failover doesn't exist
  - Eventual consistency:
    - Read repair
      - Client can detect stale responses, so write back newest response to any replicas with stale values
      - Only performed when value is read by the application, so stuff that's rarely read might be out of date
    - Anti-entropy process
      - Background process that checks for differences and fixes
  - Good for use cases that require high availability and low latency and can tolerate stale reads
- Monitoring
  - Leader-based replication: db usually has metrics for lag you can hook into
  - Leaderless: monitoring is more difficult

#### Questions

- Microsoft Azure Storage uses chain replication
- Yikes, GitHub incident when out of date follower was promoted to leader: https://github.blog/2012-09-14-github-availability-this-week/
- What is this "Mr. Poons" / "Mrs. Cake" reference from?
- What are ways to avoid conflict resolution w/ multi-leader?
- I thought Google Docs used CRDTs? I guess not. I guess it uses operational transformation.
- Counterintuitive: "For defining concurrency, exact time doesn't matter: we simply call two operations concurrent if they are both unaware of each other, regardless of the physical time at which they occurred."

#### Team discussion

### Ch 6: Partitioning

When your dataset or query load reaches a certain size, one way to maintain performance at scale is to partition your data across multiple nodes. This chapter gives us an overview of partitioning large datasets, including how to evenly distribute data and query load, properly route queries, and rebalance as you scale your cluster up/down.

Thinking back to the previous chapter, it's common to use partitioning and replication together, where you'll store copies of each partition on multiple, separate nodes. That way you'll maintain fault tolerance as you scale.

#### Partitioning of Key-Value Data

The goal of partitioning is to distribute the data and query load evenly across nodes. When unsuccessful, we end up with skewed partitioning, where a disproportionately high load is routed to one or more hot spots. We want to avoid skew, because then we miss out on all the performance benefits of partitioning (but still get all the headaches of a distributed system).

So how do we avoid skewed distribution and hot spots? The chapter describes two different approaches: key ranging and hashing.

##### Partitioning by Range

The book gave a good "real world" example of a set of encyclopedias. Each book is a partition, keys are sorted in order, and the ranges of keys can vary from book to book (for example: you may need two books to cover A-B of the encyclopedia, but only one book to cover T-Z). Partition boundaries can be set manually or automatically (more on that in the rebalancing section).

Pros:  
- Efficient lookups (provided you know the boundaries of each range + location of partition), especially range scans for related records

Cons:  
- Higher risk of skew and hot spots (example: if your data is sorted by timestamp and you're writing in real time, the partition holding most current data is overloaded)
- Need to take this into account when choosing key name

Tech that uses partitioning by range:  
- Bigtable (Google)
- HBase (open source Bigtable)
- RethinkDB
- MongoDB (< v2.4)

##### Partitioning by Hashing

Another option that's better at avoiding skew + hot spots is to partition by hash function instead. The hash function doesn't have to be strong; it just has to be consistent. For example, MD5 is good enough for MongoDB, whereas the book recommends avoiding some programming languages' built-in hash functions, since they might not have strong consistency guarantees. They give the example of Java and Ruby's built-in functions which can return a different hash value for the same key (not good for mapping data to a partition & being able to retrieve it later by key). Each partition is assigned a range of hashes, then keys are distributed evenly.

Pros:  
- More even distribution
- Less risk of skew, hot spots

Cons:  
- Range queries are less efficient, because related records are no longer going to be located near one another.
- Still susceptible to edge case-y hot spots where one key is overloaded with reads/writes (the Bieber problem)

#### Partitioning and Secondary Indexes

Secondary indexes are the "bread and butter" of relational databases, says this book, but unfortunately they introduce some additional complexity when it comes to partitioning, as you'll need to partition your indexes as well as your data. The chapter covers two main approaches: document-based partitioning and term-based partitioning.

##### Partitioning Secondary Indexes by Document

In this approach, each partition maintains its own secondary index that only includes records in that partition. It's also known as a local index.

Widely used:  
- MongoDB
- Riak
- Cassandra
- Elasticsearch
- Solrcloud
- VoltDB

Pros:  
- Isolation (don't need to worry about what's stored in other partitions)
- Simpler writes (write to one partition, update one index only on that same partition)

Cons:  
- Scatter/gather approach
- More expensive queries (need to query all partitions then combine the results)
- Prone to tail latency amplification (on your latency graph, higher peaks at the extremes)

##### Partitioning Secondary Indexes by Term

Instead of maintaining local indexes on each partition, you can use a global index partitioned by term (or more likely, a hash of the term, for more even distribution).

Pros:  
- Reads are more efficient (no scatter/gather)

Cons:  
- Writes are slower (now writes can affect multiple partitions, depending what terms are updated)
- Asynchronous updates, so index may be out of date

#### Rebalancing Partitions

As your dataset evolves, you'll probably want to adjust capacity by adding or removing nodes from your cluster. This requires you to rearrange where data is stored and requests are routed, aka rebalancing.

Goals:  
- Query and data load should be evenly distributed across partitions and nodes
- Database can still accept reads and writes during rebalancing (note: seems challenging)
- Minimize amount of data moved / requests re-routed

##### Fixed Number of Partitions

The chapter is very adamant about not partitioning based on hash mod N, because if keys are hashed based on a fixed number of nodes, then the number of nodes changes, the keys all have to be rehashed and moved, leading to excessive (and expensive) rebalancing. Instead, you could use a fixed number of partitions to start, making sure you have more partitions than nodes. Assign multiple partitions to each node, then adjust as needed when you add/remove nodes. This allows you to move partitions instead of reassigning keys.

The number of partitions is usually fixed up front when the database is originally set up and usually doesn't change. The chapter says it's difficult to choose the right number unless you have a fixed size dataset that isn't going to change.

Tech that uses this approach:  
- Riak
- Elasticsearch
- Couchbase
- Voldemort

##### Dynamic Partitioning

With dynamic partitioning, you set a max size for your partitions, then when they exceed that max, they're split into two new partitions (with data evenly divided between them). You can also do the converse and set a minimum, which will merge partitions that get too small. This approach is a better fit for key range-partitioned databases.

Tech that uses this approach:  
- HBase
- RethinkDB
- MongoDB
- Vitess
- Partitioning Proportionally to Nodes

With both dynamic and fixed partitioning, the number of partitions is unrelated to the number of nodes. Alternatively, you could a third approach and set a fixed number of partitions per node. The partitions grow as the dataset grows, so you add new nodes to reduce partition size. When a new node is added, it randomly selects a fixed number of existing partitions to split and migrate.

Tech that uses this approach:  
- Cassandra
- Ketama

#### Operations: Automatic or Manual Rebalancing

The book recommends including some sort of manual intervention, even if it's a just a human signing off on an automatically generated plan. Less convenient, but fewer surprises.

#### Request Routing

Service discovery: how do we know which node to talk to?

Three high level approaches:  
- Smart nodes: Allow clients to connect to any node (via load balancer), then nodes ping each other until eventually the right one responds, passing the reply back down the line to the client
  - Example: `gossip protocol` used by Cassandra and Riak
- Smart router: Use a "partition-aware" load balance. Requests are sent to a router first that knows where to forward each request.
- Smart client: Clients are aware of which partitions are assigned to which nodes and can ping the correct node directly.

Recommendation: use a coordination service like ZooKeeper. ZooKeeper acts as a registry, keeping track of which range is assigned to which partition, which partition is assigned to which node, and node IP address.

#### Questions

- What partitioning approaches do we use at GitHub?
- How do tools like Vitess help us with data partitioning?
  - Uses scatter/gather for queries
  - Locks writes during resharding, but allows reads (?)
- Has anyone worked with a system that used automatic rebalancing? Any horror stories?
- Does GitHub have any "Justin Bieber"-esque accounts that we need to make special data allowances for?

### Ch 7: Transactions

- Potential failures that transactions can protect against:
  - Database software or hardware failure in the middle of a write
  - Application crashes midway through series of operations
  - Network failures that cutoff application from database or one db node from another
  - Multiple clients writing to db at same time (overwriting each other)
  - Client reads partially updated data
  - Race conditions w/ weird bugs
- Transations
  - Group multiple reads and writes into one operation that rolls back everything if any step fails (all succeed or all fail)
  - Tradeoffs: sacrifice performance for guarantees
  - Still using same style introduced in 1975 by IBM System R (the first SQL db!)
- NoSQL dbs mostly abandoned transactions
- Two urban legands:
  - "transactions are the antithesis of scalability"
  - "transactions are an essential requirement for serious apps with valuable data"
- ACID: Atomicity, Consistency, Isolation, and Durability
  - Atomic transaction can be rolled back, safely retried
  - Consistency: application-specific notion of db in "good state"
    - Database can't guarantee "good state"; that's on the application
    - "the letter C doesn't really belong in ACID"
  - Isolation == serializability (academically)
    - Actual serializable isolation impacts performance
    - Oracle uses "snapshot isolation" (weaker guarantee but better perf)
    - Locks on records
  - Durability
    - Write to nonvolatile storage and write-ahead log
    - Replication
- BASE: Basically Available, Soft state, and Eventual consistency
  - aka "Not ACID"
- "Nothing that fundamentally prevents transactions in a distributed database", but it can be difficult or no-so-performant
- "Best effort" db: DB doesn't undo things on error
  - Example: Datastores with leaderless replication
- ActiveRecord doesn't retry on abort, which the book says is lame, because aborts are supposed to enable safe retries
- Weak isolation / concurrency bugs have caused lots of problems
  - Stolen bitcoin
    - https://bitcointalk.org/index.php?topic=499580
    - https://www.reddit.com/r/Bitcoin/comments/1wtbiu/how_i_stole_roughly_100_btc_from_an_exchange_and/
  - Financial audits
    - http://www.vldb.org/conf/2007/papers/industrial/p1263-jorwekar.pdf
  - Customer data corruption
    - https://www.michaelmelanson.net/transactions-the-limits-of-isolation/
- Read Committed Transaction Isolation
  - No dirty reads (only see data that's been committed)
  - No dirty writes (only overwrite data that's been committed)
  - Default for Oracle, PostgreSQL, SQL Server, MemSQL
  - Implemented via row level locks
    - Write locks are practical
    - Read locks not so much
  - Allows aborts
  - Vulnerable to "read skew", aka timing anomaly, aka nonrepeatable read
    - Bad for backups
    - Bad for long running analytics or integrity checks
- Combine read committed with "snapshot isolation" to prevent read skews
- Snapshot isolation
  - "Each transaction reads from a consistent snapshot of the database"
  - Supported by PostgreSQL, MySQL, InnoDB, Oracle, SQL Server
  - Implemented with write locks
  - No locks on reads
  - "Readers never block writers, and writers never block readers"
  - Maintains multiple versions of an object (multi-version concurrency control (MVCC))
  - Long running transaction uses same snapshot for a lonnng time
  - Referred to as "repeatable read" by SQL dbs because SQL doesn't support proper snapshot isolation. But due to naming confusion, "repeatable read" can mean different things too.
- Handling concurrent writes (clobbering)
  - Atomic write operations
  - Explicit locking
  - Auto-detect lost updates, then abort and retry
  - Compare-and-set (assumes single source of truth for data)
  - Conflict resolution
- Write Skew
  - Race condition
  - Pattern: Select query checks requirement, then application decides how to proceed, then writes... which changes results of the original check.
  - Examples
    - Booking a conference room
    - Multiplayer game (two players try to move to the same square)
    - Shared concurrent document editing
    - Claiming a username
    - Double-spending (going over your limit
- Serializable isolation
  - Strongest guarantees
  - Avoid concurrency by removing concurrency altogether and executing transactions serially
  - Limited to use cases where the active dataset can fit in memory
- Two-phase locking
  - Reads and writes block both reads and writes
  - Locks can be in shared mode or exclusive mode
    - Use shared mode for reads
    - Use exclusive mode for writes
  - Poor performance
- Serializable snapshot isolation (SSI)
  - Promising new approach
  - Optimistic concurrency control
    - No blocking; just abort on commit if necessary

#### Questions

- What does durability mean for projects like the Arctic Code Vault? Especially given all the potential faults/risks/flaws mentioned in this chapter.
- What doesn't ActiveRecord retry aborted transactions?
  - Not all operations are idempotent
  - Avoid overload (if you're not using exponential backoff)
  - Put the responsiblity on the developer to decide when to retry
- Why is it so hard to achieve ubiquitous terminology? "Nobody really knows what repeatable read means"

### Ch 8: The Trouble with Distributed Systems

- Buckle up: "the last few chapters have... been too optimistic"
- :cool-cry: "This chapter is a thoroughly pessimistic and depressing overview of things that may go wrong in a distributed system"
  - Examples:
    - Network partitions
    - Power distribution unit failures
    - Switch failures
    - Accidental power cycles
    - Data center backbone failures
    - Data center power failures
    - Humans (truck crashing into HVAC anecdote)
    - Sharks (literal sharks)
- Software on a single computer shouldn't be flaky
  - Same result every time you run a program ("deterministic")
  - Hardware problem is a total system failure; not partial
  - Fully functional or fully broken; never in between
  - Obscures / avoids messiness
- Networks are flaky
  - Nondeterministic (no guarantee you'll get the same results every time)
  - Prone to partial failures (unlike single computer)
  - Try to build a reliable higher-level system from unreliable lower-level parts (example: TCP transport layer on top of IP)
  - Shared-nothing has become dominate approach for building web services
    - "Cheap"
    - Can use cloud computing
    - High reliablity through redundant data centers
  - So many ways for network requests to fail
    - Request is lost
    - Request is waiting in a queue
    - Remote node failed
    - Remote node temporarily stopped responding (may resume in future)
    - Request was processed but response was lost
    - Request was processed but response was delayed
  - "We have been building computer networks for decades--one might hope that by now we would have figured out how to make them reliable. However, it seems that we have not yet succeeded." A+ SNARK.
  - TIL: "...just because a network link works in one direction doesn't guarantee it's also working in the opposite direction."
  - User Datagram Protocol (UDP): a connectionless protocol that works like TCP but assumes that error-checking and recovery services are not required ([source](https://www.privateinternetaccess.com/blog/tcp-vs-udp-understanding-the-difference/))
  - Phone network vs Internet network
    - Phone connection
      - Synchronous circuit for every call w/ guaranteed amt of bandwidth
      - Fixed data per frame (16 bits every 250 microseconds)
      - No queueing, so we know the maxiumum latency (aka "bounded delay")
    - TCP connection
      - Not a circuit
      - Bandwidth isn't fixed; connection uses whatever's available
      - Variable-sized data
      - Unknown latency due to queues, buffering
      - Optimized for "bursty traffic"
- Unreliable Clocks
  - Examples
    - A day may not have exactly 86,400 seconds
    - Time-of-day clocks may move backward in time
    - Time on one node may be quite different from the time on another node
  - Monotonic clock
    - Guaranteed to always move forward
    - Can't be used for durations
    - Don't need to be synced
  - "Time-of-day clock"
    - Can reset backwards
    - Synced with Network Time Protocol (NTP)
  - "Smearing": fixing drift from leap seconds
  - "Confidence interval"
    - Account for drift, latency
    - Most systems obscure the uncertainty (method names seem precise)
  - Concurrency problems
    - Last Write Wins (LWW):
      - Writes can disappear
      - Can't resolve if writes are truly concurrent
      - Machine's time of day clock might be incorrect
      - Use "logical clocks" instead for ordering events
    - Snapshot isolation
      - Hard to generate global monotonically increasing transaction id shared across partitions/nodes
        - Becomes a bottleneck
      - Use timestamps instead? As long as you're ok with uncertainty and use confidence intervals
  - Determining leader
    - Leader holds lease that expires; has to renew
    - Timing is tricky; hard to know if lease has expired
  - Tools that don't translate to distributed systems:
    - Mutexes
    - Semaphores
    - Atomic counters
    - Lock-free data structures
    - Blocking queues
  - Hard real-time systems:
    - Aircraft
    - Robots
    - Cars
    - Objects interacting with physical world in general
  - Oh that pesky garbage collector
    - Treat GC pauses like brief planned outages of a node
    - Let other nodes handle requests from clients while one node is collecting its garbage
- Truth defined by the majority (example: network quorum)
  - One node alone isn't trustworthy
  - Group consensus is more trustworthy than single node
  - Use fencing tokens as protection against leader who won't step down
- "Byzantine fault tolerance": operate correctly even if nodes are malfunctioning, not obeying, or under attack by haxx0rs
  - Examples:
    - Aerospace computers protected against radiation
      - Reminds me of bit flipping / bit squatting: http://dinaburg.org/bitsquatting.html
    - Blockchains
- Safety vs liveness
  - Safety: nothing bad happens
  - Liveness: something good eventually happens

#### Questions

- Did anyone else think the previous chapters were too optimistic? Or just less pessimistic than this chapter? (IMO the author's pessimism / realism has been pretty consistent)
- Can anyone top the "hypoglycemic driver smashing his Ford pickup truck into a DC's HVAC system" horror story?
- What is a "Clos topology"? (referenced in section about large datacenter networks near [footnote #9](http://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p183.pdf))
  - P.S. Footnote #9 is a paper titled "Jupiter Rising: A Decade of Clos Topologies...", a paper that was published the same year [Jupiter Ascending](https://www.imdb.com/title/tt1617661/) premiered. These scientists may know Clos topologies, but they have no respect for the Wachowskis.
- Does anyone have a preferred framework or language that makes working with distributed systems easier? Erlang/Elixir both come to mind
  - https://medium.com/flatiron-labs/intro-to-distributed-elixir-e8a259bcc8f6
  - https://elixirschool.com/en/lessons/advanced/otp-distribution/
- What is a quartz crystal oscillator? (device powering the clock on network machine)
- Lot of footnotes in this chapter. Are there any folks recommend reading or found particularly interesting?
  - Kyle Kingsbury, Carly Rae Jepsen and the Perils of Network Partitions @ RICON2013 [(link to video)](https://youtu.be/vFDjTd9G6Xo)
  - "Notes on Distributed Systems for Young Bloods" [working link](https://web.archive.org/web/20200220095659/https://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/)

#### Team discussion

- How on earth does geographic partitioning even work? How can these flaky networks stay in sync?
  - Think about an open source repo in Asia, with contributors from all over the world
  - Need to nominate a primary "write" machine (one writer per region)
    - Running a multi-master doesn't increase reliability
  - So your primary user record is in US datacenter, but replicated in EU datacenters for performance, etc
  - Vitess has data homing
- Spanner: time ranges give more guantees, less performance
- Logbook project
  - Conceptually a success?
  - Still active?
  - Difficulty deploying to Enterprise server
- gRPC: bi-directional streaming (heartbeat)
- Elixir
  - Supervisor similar to mySQL Orchestrator (or Zookeeper?)
  - Riak (distributed, decentralized data storage system) built on Erlang https://github.com/basho/riak
- Orchestrator failover outage: https://github.blog/2018-10-30-oct21-post-incident-analysis/
- 500 mile email: http://web.mit.edu/jemorris/humor/500-miles
- @look's [Brief History of Timekeeping](http://luke.francl.org/talks/timekeeping/)
- "Stick with boring technology": GitHub and Etsy philosophy
- ["12 Signs You're Working in a Feature Factory"](https://amplitude.com/blog/12-signs-youre-working-in-a-feature-factory-3-years-later)

### Ch 9: Consistency and Consensus



#### Questions

#### Team discussion

## Part 3: Derived Data

### Ch 10: Batch Processing

#### Questions

#### Team discussion

### Ch 11: Stream Processing

#### Questions

#### Team discussion

### Ch 12: The Future of Data Systems

#### Questions

#### Team discussion
