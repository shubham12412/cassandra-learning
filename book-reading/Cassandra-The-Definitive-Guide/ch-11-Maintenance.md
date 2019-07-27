https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch11.html

### Health Check
There are some basic things that you’ll want to look for to ensure that nodes in your cluster are healthy:

--------------------------------------------------------------------------------------------------------------------

### Basic Maintenance
There are a few tasks that you’ll need to perform before or after more impactful tasks. For example, it makes sense to take a snapshot only after you’ve performed a flush. So in this section we look at some of these basic maintenance tasks.

### Flush
To force Cassandra to write data from its memtables to SSTables on the filesystem, you use the flush command on nodetool, like this:

$ nodetool flush

You can selectively flush specific keyspaces or even specific tables within a keyspace by naming them on the command line:

$ nodetool flush hotel \
$ nodetool flush hotel reservations_by_hotel_date hotels_by_poi

Running flush also allows Cassandra to clear commitlog segments, as the data has been written to SSTables.

The nodetool drain command is similar to flush. This command actually performs a flush and then directs Cassandra to stop listening to commands from the client and other nodes. The drain command is typically used as part of an orderly shutdown of a Cassandra node and helps the node startup to run more quickly, as there is no commitlog to replay.

### cleanup
The cleanup command scans all of the data on a node and discards any data that is no longer owned by the node. You might ask why a node would have any data that it doesn’t own.

Say that you’ve had a cluster running for some time, and you want to change the replication factor or the replication strategy. If you decrease the number of replicas for any data center, then there will be nodes in that data center that no longer serve as replicas for secondary ranges.

Or perhaps you’ve added a node to a cluster and reduced the size of the token range(s) owned by each node. Then each node may contain data from portions of token ranges it no longer owns.

In both of these cases, Cassandra does not immediately delete the excess data, in case a node goes down while you’re in the middle of your maintenance. Instead, the normal compaction processes will eventually discard this data.

However, you may wish to reclaim the disk space used by this excess data more quickly to reduce the strain on your cluster. To do this, you can the nodetool cleanup command.

As with the flush command, you can select to cleanup specific keyspaces and tables. You don’t need to automate running cleanup, as you will only need to run it if you have performed one of the actions described earlier.

### Repair
As we discussed in Chapter 6, Cassandra’s tuneable consistency means that it is possible for nodes in a cluster to get out of sync over time. For example, writes at consistency levels less than ALL may succeed even if some of the nodes don’t respond, especially when a cluster is under heavy load. It’s also possible for a node to miss mutations if it is down or unreachable for longer than the time window for which hints are stored. The result is that different replicas for a different partition may have different versions of our data.

This is especially challenging when the missed mutations are deletions. A node that is down when the deletion occurs and remains offline for longer than the gc_grace_​seconds defined for the table in question can “resurrect” the data when it is brought back online.

Fortunately, Cassandra provides multiple anti-entropy mechanisms to help mitigate against inconsistency. We’ve already discussed how read repair and higher consistency levels on reads can be used to increase consistency. The final key element of Cassandra’s arsenal is the anti-entropy repair or manual repair, which we perform using the nodetool repair command.

Let’s look at what is happening behind the scenes when you run nodetool repair on a node. ***The node on which the command is run serves as the coordinator node for the request. The org.apache.cassandra.service.ActiveRepairService class is responsible for managing repairs on the coordinator node and processes the incoming request. The ActiveRepairService first executes a read-only version of a major compaction, also known as a validation compaction. During a validation compaction, the node examines its local data store and creates Merkle trees containing hash values representing the data in one of the tables under repair. This part of the process is generally expensive in terms of disk I/O and memory usage.***

***Next, the node initiates a TreeRequest/TreeResponse conversation to exchange Merkle trees with neighboring nodes. If the trees from the different nodes don’t match, they have to be reconciled in order to determine the latest data values they should all be set to. If any differences are found, the nodes stream data to each other for the ranges that don’t agree. When a node receives data for repair, it stores it in SSTables.***

Note that if you have a lot of data in a table, the resolution of Merkle trees will not go down to the individual partition. For example, in a node with a million partitions, each leaf node of the Merkle tree will represent about 30 partitions. Each of these partitions will have to be streamed even if only a single partition requires repair. This behavior is known as overstreaming. For this reason, the streaming part of the process is generally expensive in terms of network I/O, and can result in duplicate storage of data that did not actually need repair.

This process is repeated on each node, for each included keyspace and table, until all of the token ranges in the cluster have been repaired.

Although repair can be an expensive operation, Cassandra provides several options to give you flexibility in how the work is spread out.


### SEQUENTIAL AND PARALLEL REPAIR
A sequential repair works on repairing one node at a time, while parallel repair works on repairing multiple nodes with the same data simultaneously. Sequential repair was the default for releases through 2.1, and parallel repair became the default in the 2.2 release.

***When a sequential repair is initiated using the -seq option, a snapshot of data is taken on the coordinator node and each replica node, and the snapshots are used to construct Merkle trees. The repairs are performed between the coordinator node and each replica in sequence***. During sequential repairs, Cassandra’s dynamic snitch helps maintain performance. Because replicas that aren’t actively involved in the current repair are able to respond more quickly to requests, the dynamic snitch will tend to route requests to these nodes.


A parallel repair is initiated using the -par option. ***In a parallel repair, all replicas are involved in repair simultaneously, and no snapshots are needed. Parallel repair places a more intensive load on the cluster than sequential repair, but also allows the repair to complete more quickly.***


### PARTITIONER RANGE REPAIR
When you run repair on a node, by default Cassandra repairs all of the token ranges for which the node is a replica. This is appropriate for the situation where you have a single node that is in need of repair—for example, a node that has been down and is being prepared to bring back online.

However, if you are doing regular repairs for preventative maintenance, as recommended, repairing all of the token ranges for each node means that you will be doing multiple repairs over each range. For this reason, the nodetool repair command provides the -pr option, which allows you to repair only the primary token range or partitioner range. If you repair each node’s primary range, the whole ring will be repaired.

### Adding Nodes to an Existing Data Center

In most cases, you’ll want to configure your new nodes to begin bootstrapping immediately—that is, claiming token ranges and streaming the data for those ranges from other nodes. This is controlled by the autobootstrap property, which defaults to true. You can add this to your cassandra.yaml file to explicitly enable or disable auto bootstrapping.
Once the nodes are configured, you can start them, and use nodetool status to determine when they are fully initialized.

You can also watch the progress of a bootstrap operation on a node by running the nodetool bootstrap command.  If you’ve started a node with auto bootstrapping disabled, you can also kick off bootstrapping remotely at the time of your choosing with the command nodetool bootstrap resume.

After all new nodes are running, make sure to run a nodetool cleanup on each of the previously existing nodes to clear out data that is no longer managed by those nodes.

----------------------------------------------------------------------------------------------------------------------

### Adding a Data Center to a Cluster
There are several reasons you might want to add an entirely new data center to your cluster. For example, let’s say that you are deploying your application to a new data center in order to reduce network latency for clients in a new market. Or perhaps you need an active-active configuration in order to support disaster recovery requirements for your application. A third popular use case is to create a separate data center that can be used for analytics without impacting online customer transactions.


------------------------------------------------------------------------------------------------------------------------


### Summary
In this chapter, we looked at some of the ways you can interact with Cassandra to perform routine maintenance tasks; add, remove, and replace nodes; back up and recover data with snapshots, and more. We also looked at some tools that help automate these tasks to speed up maintenance and reduce errors.






