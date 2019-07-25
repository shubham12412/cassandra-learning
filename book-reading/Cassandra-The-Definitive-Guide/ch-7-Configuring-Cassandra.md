https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch07.html

### Cassandra Cluster Manager

### Creating a Cluster

***Cassandra clusters are given names in order to prevent machines in one cluster from joining another that you don’t want them to be a part of***. The name of the default cluster in the cassandra.yaml file is “Test Cluster.” You can change the name of the cluster by updating the cluster_name property—just make sure that you have done this on all nodes that you want to participate in this cluster.



### Seed Nodes
***A new node in a cluster needs what’s called a seed node. A seed node is used as a contact point for other nodes, so Cassandra can learn the topology of the cluster—that is, what hosts have what ranges***. For example, if node A acts as a seed for node B, when node B comes online, it will use node A as a reference point from which to get data. This process is known as bootstrapping or sometimes auto bootstrapping because it is an operation that Cassandra performs automatically. ***Seed nodes do not auto bootstrap because it is assumed that they will be the first nodes in the cluster.***

By default, the cassandra.yaml file will have only a single seed entry set to the localhost:

  - seeds: "127.0.0.1"
  
To add more seed nodes to a cluster, we just add another seed element. We can set multiple servers to be seeds just by indicating the IP address or hostname of the node. 

In a production cluster, these would be the IP addresses of other hosts rather than loopback addresses. ***To ensure high availability of Cassandra’s bootstrapping process, it is considered a best practice to have at least two seed nodes per data center. This increases the likelihood of having at least one seed node available should one of the local seed nodes go down during a network partition between data centers.***

------------------------------------------------------------------------------------------------------------------------


### Snitches
The job of a snitch is simply to determine relative host proximity. Snitches gather some information about your network topology so that Cassandra can efficiently route requests. The snitch will figure out where nodes are in relation to other nodes. Inferring data centers is the job of the replication strategy. You configure the endpoint snitch implementation to use by updating the endpoint_snitch property in the cassandra.yaml file.

------------------------------------------------------------------------------------------------------------------------

### Tokens and Virtual Nodes

In general, it is highly recommended to use vnodes, due to the additional burden of calculating tokens and manual configuration steps required to rebalance the cluster when adding or deleting single-token nodes.

---------------------------------------------------------------------------------------------------------------------

### Adding Nodes to a Cluster

As we’ve already discussed, to add a new node manually, we need to configure the cassandra.yaml file for the new node to set the seed nodes, partitioner, snitch, and network ports. If you’ve elected to create single token nodes, you’ll also need to calculate the token range for the new node and make adjustments to the ranges of other nodes.

----------------------------------------------------------------------------------------------------------------------

### Replication Strategies

### WHAT ARE DURABLE WRITES?
The durable_writes property allows you to bypass writing to the commit log for the keyspace. This value defaults to true, meaning that the commit log will be updated on modifications. Setting the value to false increases the speed of writes, but also has the risk of losing data if the node goes down before the data is flushed from memtables into SSTables.

***The first replica will always be the node that claims the range in which the token falls, but the remainder of the replicas are placed according to the replication strategy you use.***

### NetworkTopologyStrategy
The NetworkTopologyStrategy distributes the replicas as follows: the first replica is replaced according to the selected partitioner. Subsequent replicas are placed by traversing the nodes in the ring, skipping nodes in the same rack until a node in another rack is found. The process repeats for additional replicas, placing them on separate racks. Once a replica has been placed in each rack, the skipped nodes are used to place replicas until the replication factor has been met.

he NetworkTopologyStrategy allows you to specify a replication factor for each data center. Thus, the total number of replicas that will be stored is equal to the sum of the replication factors for each data center. 

***As a general guideline, you can anticipate that your write throughput capacity will be the number of nodes divided by your replication factor***. So in a 10-node cluster that typically does 10,000 writes per second with a replication factor of 1, if you increase the replication factor to 2, you can expect to do around 5,000 writes per second.



