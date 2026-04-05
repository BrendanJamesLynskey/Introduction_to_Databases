# Introduction to Databases

**Modern Software Architecture Series**

Relational theory | SQL fundamentals | Transactions & ACID | Indexing & query optimisation | NoSQL models | Choosing the right engine

*Mid-level software engineer track -- 22 slides*

---

## Table of Contents

1. [Why Dedicated Databases?](#slide-02--why-dedicated-databases)
2. [Database Taxonomy](#slide-03--database-taxonomy)
3. [The Relational Model](#slide-04--the-relational-model)
4. [SQL Fundamentals](#slide-05--sql-fundamentals)
5. [JOIN Types](#slide-06--join-types)
6. [Normalisation](#slide-07--normalisation)
7. [ACID Transactions](#slide-08--acid-transactions)
8. [Isolation Levels & Read Phenomena](#slide-09--isolation-levels--read-phenomena)
9. [Indexing](#slide-10--indexing)
10. [Query Planning & EXPLAIN](#slide-11--query-planning--explain)
11. [Concurrency Control](#slide-12--concurrency-control)
12. [Storage Engine Internals](#slide-13--storage-engine-internals)
13. [NoSQL -- Document Databases](#slide-14--nosql--document-databases)
14. [NoSQL -- Key-Value & Wide-Column](#slide-15--nosql--key-value--wide-column)
15. [NoSQL -- Graph & Vector Databases](#slide-16--nosql--graph--vector-databases)
16. [CAP Theorem & BASE](#slide-17--cap-theorem--base)
17. [Replication & Sharding](#slide-18--replication--sharding)
18. [Schema Design Patterns](#slide-19--schema-design-patterns)
19. [Migrations & Schema Evolution](#slide-20--migrations--schema-evolution)
20. [Choosing the Right Database](#slide-21--choosing-the-right-database)
21. [Summary & Further Reading](#slide-22--summary--further-reading)

---

## Slide 02 -- Why Dedicated Databases?

### The naive alternative

Flat files (CSV, JSON on disk) seem appealing for small data sets, but quickly expose hard limits:

- No concurrent access control -- two writers corrupt the file
- Full-file scans for every query -- `O(n)` always
- No schema enforcement -- garbage in, garbage out
- No durability guarantees -- half-written state on crash
- Ad-hoc application logic replaces reusable primitives

### What a DBMS provides

- **Persistent, structured storage** -- schema + types enforced at write time
- **Efficient retrieval** -- indexes shrink large scans to sub-millisecond lookups
- **Concurrency control** -- isolation prevents reads seeing partial writes
- **Recovery** -- write-ahead logging survives crashes with zero data loss
- **Access control** -- row/column security, roles, audit trails

---

## Slide 03 -- Database Taxonomy

### Relational (RDBMS)

Data in tables with typed columns. Relationships via foreign keys. Query via SQL.

`PostgreSQL` | `MySQL` | `SQLite` | `Oracle` | `SQL Server`

*Default choice for most transactional workloads.*

### NoSQL

Schema-flexible alternatives optimised for different data shapes or access patterns.

`Document` | `Key-Value` | `Wide-Column` | `Graph`

*MongoDB, Redis, Cassandra, Neo4j*

### NewSQL / Hybrid

ACID guarantees with horizontal scale-out -- bridging relational and distributed models.

`CockroachDB` | `Spanner` | `TiDB` | `YugabyteDB`

*Emerging default for global transactional apps.*

> **Rule of thumb:** start with a relational database. Migrate to a specialised store only when you have a concrete, measured reason -- not speculation.

---

## Slide 04 -- The Relational Model

Codd's 1970 model remains the foundation. Core concepts:

| Term | Definition |
|------|-----------|
| **Relation** | A named table -- a set of tuples sharing a schema |
| **Tuple** | One row -- an ordered list of typed attribute values |
| **Attribute** | A named column with a domain (type constraint) |
| **Primary key** | Minimal set of attributes that uniquely identify a tuple |
| **Foreign key** | Attribute referencing a primary key in another relation -- enforces referential integrity |
| **Domain** | Valid value set for an attribute: `INTEGER`, `VARCHAR(255)`, `TIMESTAMP`, ... |

> Tables are *sets* (unordered, no duplicates). SQL adds bags (multisets) for performance, but the relational spirit is set-based.

### Schema diagram

```
users                        orders                    order_items
-----------------------      -----------------------   -------------------
PK  user_id   INTEGER        PK  order_id  INTEGER     PK  item_id   INTEGER
    username  VARCHAR         FK  user_id   INTEGER     FK  order_id  INTEGER
    email     VARCHAR             total     DECIMAL         product   VARCHAR
    created   TIMESTAMP           status    VARCHAR
    role      VARCHAR             placed_at TIMESTAMP

users.user_id ──FK──> orders.user_id
orders.order_id ──FK──> order_items.order_id
```

---

## Slide 05 -- SQL Fundamentals

### Data sub-languages

| Sub-language | Statements |
|-------------|-----------|
| **DDL** -- Definition | `CREATE TABLE`, `ALTER`, `DROP` |
| **DML** -- Manipulation | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** -- Control | `GRANT`, `REVOKE` |
| **TCL** -- Transaction | `BEGIN`, `COMMIT`, `ROLLBACK` |

> SQL is *declarative* -- you describe **what** you want, not how to retrieve it. The query planner decides the execution strategy.

### Anatomy of a SELECT

```sql
SELECT
    u.username,
    COUNT(o.order_id)   AS total_orders,
    SUM(o.total)        AS lifetime_value
FROM   users  u
JOIN   orders o ON o.user_id = u.user_id
WHERE  u.role = 'customer'
  AND  o.status = 'completed'
GROUP  BY u.username
HAVING SUM(o.total) > 500
ORDER  BY lifetime_value DESC
LIMIT  20;
```

Logical evaluation order: `FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT`

---

## Slide 06 -- JOIN Types

| JOIN type | Returns |
|-----------|---------|
| `INNER JOIN` | Rows matching on both sides |
| `LEFT JOIN` | All left rows; NULLs where no right match |
| `RIGHT JOIN` | All right rows; NULLs where no left match |
| `FULL OUTER JOIN` | All rows from both; NULLs where no match |
| `CROSS JOIN` | Cartesian product -- every combination |
| `SELF JOIN` | Table joined to itself via alias |

> Most production queries use `INNER JOIN` or `LEFT JOIN`. `CROSS JOIN` on large tables can produce billions of rows -- use with intention.

**SELF JOIN example:**

```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e2.id = e1.manager_id;
```

---

## Slide 07 -- Normalisation

A process of decomposing tables to eliminate redundancy and update anomalies -- formalised as a progression of *Normal Forms* (NF).

| Normal Form | Rule |
|-------------|------|
| **1NF** | **Atomic values only.** No repeating groups, no comma-delimited lists in a single cell. Each column holds one indivisible value. |
| **2NF** | **1NF + no partial dependency.** Every non-key attribute depends on the *whole* primary key (matters for composite PKs). |
| **3NF** | **2NF + no transitive dependency.** No non-key column determines another non-key column. "Every fact depends on the key, the whole key, and nothing but the key." |
| **BCNF** | **Stricter 3NF.** Every determinant must be a superkey. Handles edge cases with overlapping candidate keys. |

### Anomalies prevented by normalisation

- **Insertion** -- can't add a new fact without inserting a full row
- **Update** -- changing a value requires updating it in many rows
- **Deletion** -- deleting the last row about an entity destroys unrelated data

> **Denormalisation** is valid -- deliberately adding redundancy to reduce joins in read-heavy workloads. Profile before denormalising; 3NF is usually fast enough.

---

## Slide 08 -- ACID Transactions

A *transaction* is a logical unit of work that must succeed entirely or have no effect at all.

| Letter | Property | Description |
|--------|----------|-------------|
| **A** | **Atomicity** | All or nothing. If any statement in the transaction fails, every preceding change is rolled back. Implemented via undo logs. |
| **C** | **Consistency** | Valid state to valid state. The transaction must satisfy all integrity constraints (NOT NULL, FK, CHECK) at commit time. |
| **I** | **Isolation** | Concurrent transactions don't see each other's partial changes. Configured via isolation levels (READ COMMITTED, REPEATABLE READ, SERIALIZABLE). |
| **D** | **Durability** | Committed data survives crashes. Achieved with write-ahead logging (WAL): changes written to a sequential log before touching the data pages. |

> `BEGIN;` ...statements... `COMMIT;` -- explicit transaction boundaries. Most ORMs wrap each statement in an implicit transaction unless you open one explicitly.

---

## Slide 09 -- Isolation Levels & Read Phenomena

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Typical Use |
|----------------|-----------|-------------------|-------------|-------------|
| `READ UNCOMMITTED` | Possible | Possible | Possible | Analytics on stale data; rarely used |
| `READ COMMITTED` | Prevented | Possible | Possible | PostgreSQL/Oracle default; most web apps |
| `REPEATABLE READ` | Prevented | Prevented | Possible | MySQL/InnoDB default; reports, exports |
| `SERIALIZABLE` | Prevented | Prevented | Prevented | Financial, inventory; highest contention |

### Read phenomena explained

- **Dirty Read** -- Reading uncommitted changes from another transaction -- never safe.
- **Non-Repeatable Read** -- Same row read twice in one transaction returns different values.
- **Phantom Read** -- A range query returns different rows across two reads in one transaction.

---

## Slide 10 -- Indexing

### Why indexes?

A table scan is `O(n)`. A B-tree index lookup is `O(log n)`. At one million rows that is ~20 comparisons vs 1,000,000.

### B-tree index (default)

Balanced tree structure. Maintains sorted order on the indexed column(s). Excellent for equality, range, prefix, and ORDER BY queries.

```
B-tree structure:

           [50 | 75]              <-- Root
          /    |     \
   [20|35]  [55|65]  [80|90]     <-- Internal nodes
   /    \    /   \    /    \
 [10] [28] [40] [52] [67] [82] [92]  <-- Leaf nodes (linked)
```

### Index types

| Index Type | Use Case |
|-----------|----------|
| **B-tree** | Equality & range; default in Postgres, MySQL |
| **Hash** | Equality only; faster for exact match, no range |
| **GIN** | Arrays, JSONB, full-text search (Postgres) |
| **GiST / SP-GiST** | Geospatial, geometric, nearest-neighbour |
| **Partial** | Indexes only rows matching a WHERE predicate -- smaller, faster |
| **Covering** | Includes all columns the query needs -- heap access avoided entirely (index-only scan) |

> **Index costs:** every write must update all indexes on that table. Over-indexing slows INSERTs/UPDATEs. Profile and add indexes in response to slow queries, not pre-emptively.

---

## Slide 11 -- Query Planning & EXPLAIN

The query planner transforms SQL into a physical execution plan, choosing between sequential scans, index scans, join algorithms (hash, merge, nested loop), and parallel workers.

```sql
-- See the plan
EXPLAIN SELECT * FROM orders
WHERE user_id = 42;

-- See actual execution times
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders
WHERE user_id = 42;
```

> **ANALYZE** actually runs the query. Use on dev/staging, not production during peak load.

### Plan node vocabulary

| Node | Description |
|------|------------|
| **Seq Scan** | Full table scan -- fine for small tables or high-selectivity queries |
| **Index Scan** | Traverse B-tree then fetch heap row for each match |
| **Index Only Scan** | All needed columns in the index -- zero heap I/O |
| **Bitmap Heap Scan** | Batch-fetch heap pages after a bitmap index pass -- good for medium selectivity |
| **Hash Join** | Build hash table on smaller side; probe with larger -- good for large equi-joins |
| **Merge Join** | Merge two sorted inputs -- efficient when both already indexed |
| **Nested Loop** | For each outer row, scan inner -- efficient when inner side is indexed and small |

---

## Slide 12 -- Concurrency Control

### Pessimistic locking (2PL)

Acquire locks before accessing data; release at commit. Guarantees serializability but risks deadlock.

```sql
-- Exclusive lock: block all other reads + writes
SELECT balance FROM accounts
WHERE id = 1
FOR UPDATE;

-- Share lock: allow other reads, block writes
SELECT balance FROM accounts
WHERE id = 1
FOR SHARE;
```

> **Deadlock** -- transaction A holds lock X and waits for Y; B holds Y and waits for X. The DBMS detects the cycle and aborts one party. Always acquire locks in a consistent order to prevent deadlocks.

### Optimistic / MVCC

*Multi-Version Concurrency Control* -- the approach used by PostgreSQL, MySQL InnoDB, and most modern RDBMSes.

- Each write creates a **new row version** with a transaction timestamp
- Readers see a **consistent snapshot** -- the latest committed version at transaction start
- No read locks -- readers never block writers and vice-versa
- Old versions are later vacuumed / purged

> MVCC makes `SELECT` essentially free in terms of locking cost. The trade-off is dead-tuple bloat -- vacuum keeps this manageable.

---

## Slide 13 -- Storage Engine Internals

### Heap files & pages

Table data stored in fixed-size *pages* (8 kB in Postgres, 16 kB in MySQL). Each page holds multiple tuples plus a page header and item ID array.

```
Page layout:

+-------------------------------------------------------+
| Page Header (lsn, checksum, free-space pointers)      |
+-------------------------------------------------------+
| Item ID array [ptr->t1] [ptr->t2] [ptr->t3] [ptr->t4]|
+-------------------------------------------------------+
|               free space                               |
+-------------------------------------------------------+
| tuple t4  (xmin=105, xmax=inf)  | 42 | Alice | 2024  |
| tuple t3  (xmin=98, xmax=105, DEAD) | 42 | Al...     |
| tuple t2  ...  tuple t1  ... (dead tuples, VACUUM)    |
+-------------------------------------------------------+
```

### Write-Ahead Log (WAL)

Changes are written to the WAL (sequential) *before* the data pages are modified. On crash, the engine replays the WAL to restore consistency.

| Term | Description |
|------|------------|
| **LSN** | Log Sequence Number -- monotonically increasing WAL position |
| **Checkpoint** | Flush dirty pages to disk; WAL before the checkpoint can be recycled |
| **REDO** | Replay committed WAL entries after crash |
| **UNDO** | Roll back uncommitted entries found in the log |

### LSM-Tree alternative

Used by RocksDB, Cassandra, ScyllaDB. Writes go to an in-memory *memtable*, then flush to immutable sorted string tables (SSTables) on disk. Sequential write throughput far exceeds B-tree for write-heavy workloads.

---

## Slide 14 -- NoSQL -- Document Databases

Store self-describing JSON/BSON documents. No fixed schema -- each document in a collection can have different fields. Exemplar: *MongoDB*, *CouchDB*, *Firestore*.

```json
{
  "_id": "ord_7821",
  "user": {
    "id": "usr_42",
    "name": "Alice",
    "tier": "gold"
  },
  "items": [
    { "sku": "GPU-4090", "qty": 1, "price": 1599.00 },
    { "sku": "RAM-32G",  "qty": 2, "price":   89.99 }
  ],
  "placed_at": "2025-03-01T14:22:00Z",
  "status": "shipped"
}
```

> The entire order and its line items live in one document -- one read to retrieve everything. Contrast with SQL: `orders` JOIN `order_items`.

### Strengths

- Matches object models directly -- less ORM impedance
- Schema evolution is easy -- add fields without migrations
- Horizontal sharding by document key is natural
- Rich query support (aggregation pipelines, text search)

### Weaknesses

- No cross-document joins in a single atomic operation (in most systems)
- Data duplication if same sub-document embedded in many places
- ACID across documents only in recent versions (MongoDB 4.x+)
- Flexible schema is a double-edged sword -- discipline needed

---

## Slide 15 -- NoSQL -- Key-Value & Wide-Column

### Key-Value stores

Opaque value blob addressed by a key. Fastest possible read/write. *Redis*, *DynamoDB*, *etcd*.

```
SET session:u42 "{user_id:42, cart:[...]}" EX 3600
GET session:u42
DEL session:u42
```

Redis extends the model with typed values: strings, hashes, lists, sets, sorted sets, streams, and HyperLogLog.

- **Best for:** caching, sessions, pub/sub, leaderboards, rate limiting
- **Weakness:** no ad-hoc querying beyond key lookup; value is opaque to the engine

### Wide-Column stores

Rows identified by a partition key; columns grouped into column families; sparse -- not all rows have all columns. *Cassandra*, *HBase*, *Bigtable*.

```cql
CREATE TABLE events (
  device_id  UUID,
  ts         TIMESTAMP,
  temp_c     FLOAT,
  PRIMARY KEY (device_id, ts)
) WITH CLUSTERING ORDER BY (ts DESC);
```

- **Best for:** time-series, IoT, write-heavy append workloads at massive scale
- **Weakness:** data model must be designed around query patterns; no ad-hoc joins; eventual consistency by default

---

## Slide 16 -- NoSQL -- Graph & Vector Databases

### Graph databases

Nodes and edges with arbitrary properties. Traversal queries that would require many recursive self-joins in SQL are first-class operations. *Neo4j*, *Amazon Neptune*, *TigerGraph*.

```cypher
// Who follows Alice's followers?
MATCH (a:User {name:"Alice"})
      <-[:FOLLOWS]-(b:User)
      <-[:FOLLOWS]-(c:User)
RETURN DISTINCT c.name
LIMIT 20;
```

**Best for:** social graphs, fraud detection, recommendation engines, knowledge graphs, network topology.

### Vector databases

Store high-dimensional embedding vectors and answer approximate nearest-neighbour (ANN) queries. Indexing algorithms: HNSW, IVF-Flat, ANNOY. *Pinecone*, *Weaviate*, *pgvector* (Postgres extension), *Qdrant*.

```python
results = collection.query(
    query_embeddings=[embed("cat playing piano")],
    n_results=5
)
```

**Best for:** semantic search, RAG pipelines for LLMs, image/audio similarity, recommendation by content.

> Not a replacement for a relational DB -- commonly used alongside one. Store metadata in Postgres; store the vector in pgvector or a dedicated engine.

---

## Slide 17 -- CAP Theorem & BASE

In a distributed system, the CAP theorem states you can guarantee at most two of three properties simultaneously.

| Property | Description |
|----------|------------|
| **Consistency** | All nodes see the same data at the same time |
| **Availability** | Every request receives a response |
| **Partition Tolerance** | System continues operating despite network partitions |

### CAP trade-offs

| Combination | Examples |
|------------|---------|
| **CP** -- Consistency + Partition tolerance | HBase, Zookeeper |
| **AP** -- Availability + Partition tolerance | Cassandra, CouchDB |
| **CA** -- Consistency + Availability | Traditional RDBMS (single node) |

> **P is usually non-negotiable.** Network partitions happen in any distributed system. The real choice is C vs A during a partition.

### BASE vs ACID

| Property | Description |
|----------|------------|
| **Basically Available** | System stays up even during partial failure |
| **Soft state** | Data may be in flux -- not yet propagated to all replicas |
| **Eventually consistent** | Given no new updates, all replicas converge to the same value |

> DynamoDB, Cassandra, and most KV stores are BASE by default. Most support tunable consistency -- you can opt into stronger guarantees at higher latency cost.

---

## Slide 18 -- Replication & Sharding

### Replication

Maintain copies of data on multiple nodes for fault tolerance and read scaling.

| Strategy | Description |
|----------|------------|
| **Primary-Replica** | One writable primary; replicas stream the WAL and serve reads. Postgres streaming replication, MySQL binlog. |
| **Synchronous** | Commit only after replica acknowledges -- zero data loss, higher latency |
| **Asynchronous** | Commit immediately -- lower latency, small replication lag window |
| **Multi-primary** | Multiple writable nodes; conflict resolution required (Galera, Spanner) |

### Sharding (horizontal partitioning)

Split data across multiple nodes so no single node holds the full dataset.

| Strategy | Description |
|----------|------------|
| **Range sharding** | Rows with key in [0, 1M) -> shard 0; [1M, 2M) -> shard 1. Simple but hotspot-prone on sequential keys. |
| **Hash sharding** | shard = hash(key) % n. Uniform distribution; cross-shard range queries are expensive. |
| **Consistent hashing** | Ring-based; adding a node rebalances only ~1/n of keys. Used by Cassandra, DynamoDB. |

> Sharding dramatically complicates joins, transactions, and schema migrations. Exhaust vertical scaling and read replicas before sharding.

---

## Slide 19 -- Schema Design Patterns

### OLTP schema

Highly normalised (3NF). Many small tables. Optimised for fast, concurrent point writes/reads. Short transactions.

*e-commerce, banking, SaaS apps*

### Star schema (OLAP)

Central *fact table* (events, transactions) surrounded by *dimension tables* (time, product, customer). Denormalised for fast aggregation.

```
fact_sales
  -> dim_date
  -> dim_product
  -> dim_customer
```

### EAV pattern

Entity-Attribute-Value: a generic `(entity_id, attr_name, value)` triple table -- flexible but slow and hard to constrain. Use JSONB or a document DB instead in modern systems.

### Polymorphic associations

One FK column that can reference multiple tables by adding a `type` discriminator column. Efficient but violates strict referential integrity. Works well in practice for comment/like/notification systems.

### Soft delete

Add `deleted_at TIMESTAMP NULL` instead of issuing `DELETE`. Preserves audit history and enables undo. Requires `WHERE deleted_at IS NULL` on all queries -- consider a partial index or a view.

---

## Slide 20 -- Migrations & Schema Evolution

### Version-controlled migrations

Store schema changes as ordered migration files in source control. Tools: *Flyway*, *Liquibase*, *Alembic* (Python), *golang-migrate*, Rails *ActiveRecord migrations*.

```sql
-- V004__add_tier_to_users.sql
ALTER TABLE users
  ADD COLUMN tier VARCHAR(20)
    NOT NULL DEFAULT 'standard';

CREATE INDEX idx_users_tier ON users(tier);
```

> Each migration is applied exactly once. The tool tracks the current schema version in a `schema_history` / `flyway_schema_history` table.

### Zero-downtime migration patterns

| Pattern | Description |
|---------|------------|
| **Expand-contract** | 1. Add new column (nullable). 2. Deploy app writing both old & new. 3. Backfill. 4. Make NOT NULL. 5. Remove old column. |
| **Ghost / pt-osc** | Tools that rebuild a table in the background and swap atomically -- safe for large tables in production. |
| **Feature flags** | Gate new schema paths in application code; decouple deploy from schema cutover. |

> **Never run DDL in a transaction without knowing your DB's behaviour.** Postgres holds an `ACCESS EXCLUSIVE` lock for the duration of some `ALTER TABLE` statements, blocking all reads and writes.

---

## Slide 21 -- Choosing the Right Database

| Workload / Need | Primary choice | Secondary / specialist |
|-----------------|---------------|----------------------|
| Transactional CRUD (web app, API backend) | **PostgreSQL** / MySQL | SQLite (embedded / testing) |
| Analytical queries, large aggregations | **ClickHouse** / DuckDB | BigQuery, Redshift, Snowflake |
| Caching & session storage | **Redis** | Memcached, DragonflyDB |
| Flexible / schema-less documents | **MongoDB** | Postgres JSONB (avoid a second system) |
| Time-series (metrics, IoT, monitoring) | **TimescaleDB** / InfluxDB | Prometheus (metrics), QuestDB |
| Write-heavy, globally distributed | **Cassandra** / ScyllaDB | DynamoDB, CockroachDB |
| Graph traversal (social, fraud, reco) | **Neo4j** | Amazon Neptune, Memgraph |
| Semantic / vector search, RAG | **pgvector** (Postgres ext) | Pinecone, Qdrant, Weaviate |
| Full-text search | **Elasticsearch** / OpenSearch | Postgres `tsvector`, Typesense, Meilisearch |

> Before adding a new database to your stack, ask: can Postgres do this well enough? The operational overhead of a second persistence system is almost always underestimated.

---

## Slide 22 -- Summary & Further Reading

### Key takeaways

- The relational model remains the correct default -- use it unless you have a measured reason not to
- ACID transactions are not free but are far less expensive than debugging consistency bugs
- Index on measured slow queries; over-indexing degrades write throughput
- EXPLAIN ANALYZE is your most important query-tuning tool
- MVCC makes reads non-blocking; vacuum keeps dead tuples manageable
- CAP and BASE describe distributed trade-offs -- most apps sit on a single-region primary replica, making strict ACID accessible
- Migrations should be version-controlled, automated, and tested before production
- Postgres can handle documents (JSONB), time-series (TimescaleDB), vectors (pgvector), and full-text search -- one system often beats four

### Recommended reading

| Source | Description |
|--------|------------|
| **Kleppmann** | *Designing Data-Intensive Applications* -- the canonical distributed-systems and database reference |
| **Ramakrishnan** | *Database Management Systems* -- rigorous foundations: query optimisation, concurrency, recovery |
| **PostgreSQL Docs** | [postgresql.org/docs](https://www.postgresql.org/docs/) -- the best-written database manual in existence |
| **use-the-index-luke** | [use-the-index-luke.com](https://use-the-index-luke.com) -- indexing and query optimisation deep-dives |
| **CMU 15-445** | Andy Pavlo's Database Systems course -- free on YouTube, outstanding breadth |
