https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch08.html


### Clusters and Contact Points
To start building our application, we’ll use the driver’s API to connect to our cluster. In the Java driver, this is represented by the com.datastax.driver.core.Cluster and Session classes.

The Cluster class is the main entry point of the driver. It supports a fluent-style API using the builder pattern. For example, the following lines create a connection to a node running on the local host:

------------------------------------------------------------------------------------------------------------------

### PROTOCOL VERSION
The driver supports multiple versions of the CQL native protocol. Cassandra 3.0 supports version 4, as we learned in our overview of Cassandra’s release history in Chapter 2. 

------------------------------------------------------------------------------------------------------------------

### COMPRESSION
The driver provides the option of compressing messages between your client and Cassandra nodes, taking advantage of the compression options supported by the CQL native protocol. Enabling compression reduces network bandwidth consumed by the driver, at the cost of additional CPU usage for the client and server

Currently there are two compression algorithms available, LZ4 and SNAPPY, as defined by the com.datastax.driver.core.ProtocolOptions.Compression enumeration. The compression defaults to NONE but can be overridden by calling the Cluster.Builder.withCompression() operation.

------------------------------------------------------------------------------------------------------------------

### AUTHENTICATION AND ENCRYPTION
The driver provides a pluggable authentication mechanism that can be used to support a simple username/password login, or integration with other authentication systems

------------------------------------------------------------------------------------------------------------------

### Sessions and Connection Pooling
After we create our Cluster instance, it is not connected to any Cassandra nodes until we initialize it by calling the init() method:

cluster.init();

When this method is invoked, the driver connects to one of the configured contact points in order to obtain metadata about the cluster. This operation will throw a NoHostAvailableException if none of the contact points is available, or an AuthenticationException if authentication fails.


***Once we have initialized our Cluster object, we need to establish a session in order to formulate our queries***. We can obtain a com.datastax.driver.core.Session object by calling one of the Cluster.connect() operations. You can optionally provide the name of a keyspace to connect to, as we do in this example that connects to the hotel keyspace:

Session session = cluster.connect("hotel");

There is also a connect() operation with no parameters, which creates a Session that can be used with multiple keyspaces. If you choose this option, you’ll have to qualify every table reference in your queries with the appropriate keyspace name. Note that it is not strictly required to call Cluster.init() explicitly, as it is also invoked behind the scenes when we call connect().

***Each Session manages connections to a Cassandra cluster, which are used to execute queries and control operations using the Cassandra native protocol. The session contains a pool of TCP connections for each host.***

### SESSIONS ARE EXPENSIVE
***Because a session maintains connection pools for multiple nodes, it is a relatively heavyweight object.  In most cases, you’ll want to create a single Session and reuse it throughout your application, rather than continually building up and tearing down Sessions***. Another acceptable option is to create a Session per keyspace, if your application is accessing multiple keyspaces.


***Because the CQL native protocol is asynchronous, it allows multiple simultaneous requests per connection; the maximum is 128 simultaneous requests in protocol v2, while v3 and v4 allow up to 32,768 simultaneous requests. Because of this larger number of simultaneous requests, fewer connections per node are required. In fact, the default is a single connection per node.***


The driver supports the ability to scale the number of connections up or down based on the number of requests per connection. These connection pool settings are configurable via the PoolingOptions class, which sets the maximum and minimum (or “core”) number of connections to use for local and remote hosts. If the core and maximum values are different, the driver scales the size of the connection pool for each node up or down depending on the amount of requests made by the client. The settings of minimum and maximum thresholds of requests per connection are used to determine when new connections are created, and when underused connections can be reclaimed. There is also a buffer period to prevent the continual building up and tearing down of connections.

------------------------------------------------------------------------------------------------------------------

### Statements
Up until this point, we have only configured our connection to the cluster, and haven’t yet performed any reads or writes. To begin doing some real application work, we’ll create and execute statements using the com.datastax.driver.core.Statement class and its various subclasses. Statement is an abstract class with several implementations, including SimpleStatement, PreparedStatement, BoundStatement, BatchStatement, and BuiltStatement.

The simplest way to create and execute a statement is to call the Session.execute() operation with a string representing the statement. Here’s an example of a statement that will return the entire contents of our hotels table:

session.execute("SELECT * from hotel.hotels");


-------------------------------------------------------------------------------------------------------------





