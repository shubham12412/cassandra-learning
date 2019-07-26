https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch09.html

Now that we understand the data model and how to use a simple client, we’ll dig deeper into the different kinds of queries you can perform in Cassandra to read and write data. ***We’ll also take a look behind the scenes to see how Cassandra handles your read and write queries.***

----------------------------------------------------------------------------------------------------------------

### Writing
Let’s start by noting some basic properties of writing data to Cassandra. First, ***writing data is very fast in Cassandra, because its design does not require performing disk reads or seeks***. The ***memtables and SSTables save Cassandra from having to perform these operations on writes, which slow down many databases. All writes in Cassandra are append-only.***

***Because of the database commit log and hinted handoff design, the database is always writable, and within a column family, writes are always atomic.***

----------------------------------------------------------------------------------------------------------------

### Write Consistency Levels
Cassandra’s tuneable consistency levels mean that you can specify in your queries how much consistency you require on writes. ***A higher consistency level means that more replica nodes need to respond, indicating that the write has completed. Higher consistency levels also come with a reduction in availability, as more nodes must be operational for the write to succeed.***

----------------------------------------------------------------------------------------------------------------

###  The Cassandra Write Path
The write path describes how data modification queries initiated by clients are processed, eventually resulting in the data being stored on disk. We’ll examine the write path both in terms of interactions between nodes, and the internal process of storing data on an individual node.

The write path begins when a client initiates a write query to a Cassandra node which serves as the coordinator for this request. The coordinator node uses the partitioner to identify which nodes in the cluster are replicas, according to the replication factor for the keyspace. The coordinator node may itself be a replica, especially if the client is using a token-aware driver. If the coordinator knows that there are not enough replicas up to satisfy the requested consistency level, it returns an error immediately.

Next, the coordinator node sends simultaneous write requests to all replicas for the data being written. This ensures that all nodes will get the write as long as they are up. Nodes that are down will not have consistent data, but they will be repaired via one of the anti-entropy mechanisms: hinted handoff, read repair, or anti-entropy repair.


![Interactions-between-nodes-on-the-write-path.png](./img/Interactions-between-nodes-on-the-write-path.png)

If the cluster spans multiple data centers, the local coordinator node selects a remote coordinator in each of the other data centers to coordinate the write to the replicas in that data center. Each of the remote replicas responds directly to the original coordinator node.

The coordinator waits for the replicas to respond. Once a sufficient number of replicas have responded to satisfy the consistency level, the coordinator acknowledges the write to the client. If a replica doesn’t respond within the timeout, it is presumed to be down, and a hint is stored for the write. A hint does not count as successful replica write unless the consistency level ANY is used.


###  interactions that take place within each replica node to process a write request

![Interactions-within-a-node-on-the-write-path.png](./img/Interactions-within-a-node-on-the-write-path.png)


First, the replica node receives the write request and immediately writes the data to the commit log. Next, the replica node writes the data to a memtable. If row caching is used and the row is in the cache, the row is invalidated.

If the write causes either the commit log or memtable to pass their maximum thresholds, a flush is scheduled to run.

At this point, the write is considered to have succeeded and the node can reply to the coordinator node or client.

After returning, the node executes a flush if one was scheduled. The contents of each memtable are stored as SSTables on disk and the commit log is cleared. After the flush completes, additional tasks are scheduled to check if compaction is needed and then a compaction is performed if necessary.

----------------------------------------------------------------------------------------------------------------

### Writing Files to Disk

### COMMIT LOG FILES
Cassandra writes commit logs to the filesystem as binary files. The commit log files are found under the $CASSANDRA_HOME/data/commitlog directory.

Commit log files are named according to the pattern CommitLog-<version>-<timestamp>.log. For example: CommitLog-6-1451831400468.log. The version is an integer representing the commit log format. For example, the version for the 3.0 release is 6. You can find the versions in use by release in the org.apache​.cassandra​.db.commitlog.CommitLogDescriptor class.
  
### SSTABLE FILES
***When SSTables are written to the filesystem during a flush, there are actually several files that are written per SSTable.*** Let’s take a look at the $CASSANDRA_HOME/data/data directory to see how the files are organized on disk.


