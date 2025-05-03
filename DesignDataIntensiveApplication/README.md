# Data Structures That Power Your Database

**Conclusion: Comparing B-Trees and LSM-Trees**

B-Trees and LSM-Trees represent two fundamentally different approaches to indexing and data persistence, each with its own strengths and trade-offs. While B-Trees offer mature, predictable performance—especially for read-heavy workloads and transactional systems—LSM-Trees shine in write-intensive scenarios due to their efficient use of sequential writes and potential for lower write amplification. However, LSM-Trees come with complexity in managing compaction and can exhibit latency spikes under high load. Ultimately, the choice between them depends on the workload characteristics and system requirements. Empirical testing is essential, as performance varies widely with configuration and data access patterns.

#### 🔄 **Trade-offs: B-Trees vs. LSM-Trees**

***

**✅ Advantages of LSM-Trees**

1. **Higher Write Throughput**
   * Writes are sequential and buffered via memtables and WAL, making them faster than random writes in B-Trees.
   * Especially beneficial on magnetic drives and SSDs with log-structured firmware.
2. **Lower Write Amplification (Potentially)**
   * Though compaction rewrites data, properly tuned LSMs can have lower amplification than B-Trees that rewrite pages multiple times.
3. **Better Compression**
   * SSTables are not page-oriented and are periodically compacted, removing fragmentation and enabling more aggressive compression.
4. **Optimized for Disk Bandwidth**
   * Sequential disk access allows more efficient use of available I/O, especially during high write activity.
5. **Scales Well with Write-Heavy Workloads**
   * Well-suited for time-series data, logging systems, and systems with append-mostly access patterns.

***

**❌ Disadvantages of LSM-Trees**

1. **Read Amplification**
   * Reads may need to search across multiple SSTables at different compaction levels.
   * Bloom filters mitigate this, but not eliminate it entirely.
2. **Compaction Overhead**
   * Background compaction can compete with active reads/writes, causing spikes in latency or degraded performance under heavy load.
3. **Complexity in Tuning**
   * Requires careful configuration of compaction strategies (tiered vs. leveled), memory allocation, and disk management.
4. **Potential for Write Stall or Disk Overflow**
   * If compaction can't keep up with write load, unmerged segments grow, increasing read cost and risking disk overflow.
5. **Multiple Versions of Same Key**
   * Same key may exist in different segments until compaction resolves them, making concurrency control or transactions trickier to implement.

***

**✅ Advantages of B-Trees**

1. **Efficient Point Reads**
   * Keys exist in one location—reduces lookup time and simplifies indexing logic.
2. **Stable Latency**
   * Predictable read/write performance without background compaction interference.
3. **Strong Transactional Semantics**
   * Easier to implement range locks and maintain isolation levels due to singular key placement.
4. **Mature and Proven**
   * Supported in almost every traditional RDBMS, well-understood by practitioners, battle-tested.

***

**❌ Disadvantages of B-Trees**

1. **Write Amplification**
   * Pages must be written even for small updates; tree rebalancing adds further overhead.
2. **Random Writes**
   * Less efficient on HDDs/SSDs due to frequent non-sequential writes.
3. **Fragmentation**
   * Page splits and partial page usage lead to wasted space and poor compression potential.
4. **Less Adapted to Write-Heavy Workloads**
   * Struggles under high ingestion rates unless heavily optimized or batched.

***

#### 🔍 **When to Choose What**

| Scenario                                           | Best Fit      |
| -------------------------------------------------- | ------------- |
| High write throughput, log-style ingestion         | **LSM-Trees** |
| Low-latency reads and stable performance           | **B-Trees**   |
| Limited disk I/O and bandwidth                     | **LSM-Trees** |
| Strong transactional isolation needed              | **B-Trees**   |
| Workload with frequent updates to existing records | **B-Trees**   |





#### 🧩 **Popular Databases and Their Indexing/Persistence Strategies**

Here’s a list of common databases and which model they use:

| Database                   | Type                       | Primary Storage/Index Structure                                                                       |
| -------------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------- |
| **PostgreSQL**             | RDBMS                      | **B-Tree** (default), also supports GiST, GIN, BRIN, Hash                                             |
| **MySQL (InnoDB)**         | RDBMS                      | **B+Tree** (for clustered index and secondary indexes)                                                |
| **Oracle**                 | RDBMS                      | **B+Tree** (default), also supports bitmap indexes                                                    |
| **DB2**                    | RDBMS                      | **B+Tree**, and **encoded vector indexes** (EVI)                                                      |
| **SQL Server**             | RDBMS                      | **B+Tree** (called "clustered/non-clustered indexes")                                                 |
| **CockroachDB**            | NewSQL (distributed RDBMS) | **MVCC + B-Tree** (based on RocksDB internally which uses LSM, but exposed via B-tree-like semantics) |
| **Cassandra**              | NoSQL (wide-column)        | **LSM-Tree** (via SSTables + memtables + compaction)                                                  |
| **MongoDB**                | NoSQL (document)           | **B-Tree** (based on WiredTiger)                                                                      |
| **HBase**                  | NoSQL (wide-column)        | **LSM-Tree** (inspired by Bigtable)                                                                   |
| **ClickHouse**             | OLAP                       | Not B-tree or LSM — uses **MergeTree** and columnar storage                                           |
| **Elasticsearch**          | Search Engine              | Inverted index + **LSM-Tree**-like segments                                                           |
| **InfluxDB / TimescaleDB** | Time-series                | InfluxDB: **LSM-Tree-like**, Timescale: built on **PostgreSQL (B-Tree)**                              |
