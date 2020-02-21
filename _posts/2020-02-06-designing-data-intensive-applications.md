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
- Possible to use JavaScript with some SQL dbs?! https://blog.heroku.com/javascript_in_your_postgres
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
- Ink and Switch Lab
  - Conflict free collaboration with shared data
  - Immutable logs you can rebuild data structures from
- Multi-dimensional indices!
- Geospatial databases do a lot of different things than dbs discussed in this chapter. Hard to handle > 20 indexes.
- Search is a one dimensional index
  - You can trick it into thinking it's 2D: https://en.wikipedia.org/wiki/Hilbert_curve
  - "Space filling curves"
- Lucene talk about GIS geospatial stuff
  - Overload those concepts for search

### Ch 4: Encoding and Evolution

#### Questions

#### Team discussion

## Part 2: Distributed Data

### Ch 5: Replication

#### Questions

#### Team discussion

### Ch 6: Partitioning

#### Questions

#### Team discussion

### Ch 7: Transactions

#### Questions

#### Team discussion

### Ch 8: The Trouble with Distributed Systems

#### Questions

#### Team discussion

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
