https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch10.html


Cassandra also features built-in support for Java Management Extensions (JMX), which offers a rich way to monitor your Cassandra nodes and their underlying Java environment. Through JMX, we can see the health of the database and ongoing events, and even interact with it remotely to tune certain values. JMX is an important part of Cassandra.

### Logging
The simplest way to get a picture of what’s happening in your database is to just change the logging level to make the output more verbose. This is great for development and for learning what Cassandra is doing under the hood.

Cassandra uses the Simple Logging Facade for Java (SLF4J) API for logging, with Logback as the implementation.  SLF4J provides a facade over various logging frameworks such as Logback, Log4J, and Java’s built-in logger (java.util.logging)

### Monitoring Cassandra with JMX
In this section, we explore how Cassandra makes use of Java Management Extensions (JMX) to enable remote management of your servers. JMX started as Java Specification Request (JSR) 160 and has been a core part of Java since version 5.0.

***JMX is a Java API that provides management of applications in two key ways. First, JMX allows you to understand your application’s health and overall performance in terms of memory, threads, and CPU usage—things that are generally applicable to any Java application. Second, JMX allows you to work with specific aspects of your application that you have instrumented.***

------------------------------------------------------------------------------------------------------------------------

### Monitoring with nodetool

nodetool ships with Cassandra and can be found in <cassandra-home>/bin. This is a command-line program that offers a rich array of ways to look at your cluster, understand its activity, and modify it. nodetool lets you get limited statistics about the cluster, see the ranges each node maintains, move data from one node to another, decommission a node, and even repair a node that’s having trouble.
  
 ### OVERLAP OF NODETOOL AND JMX
Many of the tasks in nodetool overlap with functions available in the JMX interface. This is because, behind the scenes, nodetool is invoking JMX using a helper class called org.apache​.cassandra​.tools.NodeProbe. So JMX is doing the real work, the NodeProbe class is used to connect to the JMX agent and retrieve the data, and the NodeCmd class is used to present it in an interactive command-line interface.

### CONNECTING TO A SPECIFIC NODE
With the exception of the help command, nodetool must connect to a Cassandra node in order to access information about that node or the cluster as a whole.

You can use the -h option to identify the IP address of the node to connect to with nodetool. If no IP address is specified, the tool attempts to connect to the default port on the local machine, which is the approach we’ll take for examples in this chapter.

### Getting Cluster Information

1) DESCRIBECLUSTER \
bin/nodetool describecluster

2) STATUS \
bin/nodetool status

3) RING \
bin/nodetool ring

### Getting Statistics

4) USING TPSTATS \
The tpstats tool gives us information on the thread pools that Cassandra maintains. \

bin/nodetool tpstats

5) USING TABLESTATS \
bin/nodetool tablestats hotel





