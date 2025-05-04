# Replication

### Leader-Based Replication (Active/Passive or Master–Slave Replication)

In leader-based replication, **writes are directed to a single node (leader)**, while **reads can be served by either the leader or its replicas (followers)**.

***

#### 1.1 Synchronous vs Asynchronous Replication

* **Synchronous Replication:**
  * At least one follower must acknowledge the write.
  * Ensures durability and consistency: if the leader fails, a synced follower has all the data.
  * **Drawback:** If the synchronous follower is down or slow, the leader **blocks all writes** until it becomes available.
  * Usually, only **one follower is configured as synchronous**, and others are asynchronous. This is sometimes referred to as **semi-synchronous replication**.
* **Asynchronous Replication:**
  * Leader does **not wait** for any acknowledgment from followers before confirming write success to the client.
  * **Advantage:** No write latency or blockage due to slow followers.
  * **Risk:** If the leader fails before the follower has caught up, **some acknowledged writes may be lost**.
  * Common in setups with many followers or **geo-distributed replicas**.

***

#### 1.2 Setting Up New Followers

* New followers must start with a **consistent snapshot** of the leader’s data.
* Ideally, this snapshot is taken **without a full DB lock** to avoid disruption.
* Use tools/automation like:
  * **`pg_basebackup`** (PostgreSQL)
  * **`innobackupex`** (MySQL)

***

#### 1.3 Handling Node Outages

**A. Follower Failure: Catch-up Recovery**

* Followers keep a **replication log** of changes on local disk.
* On restart, the follower **asks the leader** for all changes since the last known offset using this log.
* Allows efficient recovery without full data re-sync.

**B. Leader Failure: Failover Process**

1. **Failure Detection:**
   * Nodes exchange heartbeats. If no response for a threshold (e.g., 30 seconds), the leader is assumed to be **dead**.
2. **Electing a New Leader:**
   * Chosen based on the **most up-to-date replica** (closest in replication log to the failed leader).
   * Caution: promoting a follower **without the latest data** may lead to inconsistencies.
3. **System Reconfiguration:**
   * Clients and nodes must redirect write requests to the new leader.
   * If the **old leader comes back**, it must **become a follower** and synchronize with the new leader.
4. **Failure Case Study:**
   * In one GitHub incident, a follower was promoted to leader without the latest data.
   * As a result:
     * IDs were **reissued** to different users.
     * Users saw **data belonging to others**.
   * To prevent such issues, some systems implement **safety mechanisms** that shut down one node if two leaders are detected.
     * ⚠️ Poorly designed mechanisms can lead to **both nodes shutting down**.

***

#### 1.4 Replication Log Implementation Strategies

**1.4.1 Statement-Based Replication (Legacy)**

* Replicates SQL statements.
* Problematic for non-deterministic operations:
  * Example: `NOW()` returns different values on different replicas.
* Requires caution or hardcoded values to ensure consistency.
* **Used in MySQL pre-v5.1**.

**1.4.2 Write-Ahead Log (WAL) Shipping**

* Used in **PostgreSQL, Oracle**, etc.
* The WAL logs **low-level byte changes** to disk blocks.
* **Tightly coupled** with the storage engine.
* Drawback:
  * Incompatible across different DB versions due to storage format changes.
  * Requires **same DB version** on leader and followers.

**1.4.3 Logical (Row-Based) Log Replication**

* Replicates **logical data changes** like "insert row X with value Y".
* **Decoupled from physical storage format**.
* More flexible and **cross-version compatible**.
* Suitable for systems needing loosely coupled replication mechanisms.

**1.4.4 Trigger-Based Replication**

* Triggers are defined on tables to capture changes.
* **More flexible**, allowing custom replication logic.
* **Downside:** Adds overhead and complexity to database operations.
