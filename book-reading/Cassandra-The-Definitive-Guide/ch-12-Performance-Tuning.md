https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch12.html

we look at how to tune Cassandra to improve performance. ***There are a variety of settings in the configuration file and on individual tables. Although the default settings are appropriate for many use cases, there might be circumstances in which you need to change them. In this chapter, we look at how and why to make these changes.***

We also see how to use the ***cassandra-stress test tool that ships with Cassandra to generate load against Cassandra and quickly see how it behaves under stress test circumstances***. We can then tune Cassandra appropriately and feel confident that we’re ready to deploy to a production environment.

### Managing Performance

### Setting Performance Goals
Before beginning any performance tuning effort, it is important to have clear goals in mind, whether you are just getting started on deploying an application on Cassandra, or maintaining an existing application.

When planning a cluster, it’s important to understand how the cluster will be used: the number of clients, intended usage patterns, expected peak hours, and so on. This will be useful in planning the initial cluster capacity and for planning cluster growth


***An important part of this planning effort is to identify clear performance goals in terms of both throughput (the number of queries serviced per unit time) and latency (the time to complete a given query).***


***For example, let’s say that we’re building an ecommerce website that uses the hotel data model we designed in Chapter 5. We might set a performance goal such as the following for shopping operations on our cluster:
The cluster must support 30,000 read operations per second from the available_rooms_by_hotel_date table with a 95th percentile read latency of 3 ms.***

This is a statement that includes both ***throughput and latency goals***. We’ll learn how to measure this using nodetool tablestats.


***Regardless of your specific performance goals, it’s important to remember that performance tuning is all about trade-offs. Having well-defined performance goals will help you articulate what trade-offs are acceptable for your application***. 


### Monitoring Performance
As the size of your cluster grows, the number of clients increases, and more keyspaces and tables are added, the demands on your cluster will begin to pull in different directions. Taking frequent baselines to measure the performance of your cluster against its goals will become increasingly important.


We learned in Chapter 10 about the various metrics that are exposed via JMX, including performance-related metrics for Cassandra’s StorageProxy and individual tables.  In that chapter, we also examined nodetool commands that publish performance-related statistics such as nodetool tpstats and nodetool tablestats and discussed how these can help identify loading and latency issues.

Now we’ll look at two additional nodetool commands that present performance statistics formatted as histograms: proxyhistograms and tablehistograms. First, let’s examine the output of the nodetool proxyhistograms command:

While the view provided by proxyhistograms is useful for identifying general performance issues, we’ll frequently need to focus on performance of specific tables. This is what nodetool tablehistograms allows us to do.


***The output shows the latency of reads, writes, and range requests for which the requested node has served as the coordinator***. The output is expressed in terms of percentile rank as well as minimum and maximum values in microseconds. ***Running this command on multiple nodes can help identify the presence of slow nodes in the cluster***. A large range latency (in the hundreds of milliseconds or more) can be an indicator of clients using inefficient range queries, such as those requiring the ALLOW FILTERING clause or index lookups. 


The output of this command is similar. It omits the range latency statistics and instead provides counts of SSTables read per query. The partition size and cell count are provided, and this provides another way of identifying large partitions.


### RESETTING METRICS
Note that in Cassandra releases through the 3.X series, the metrics reported are lifetime metrics since the node was started. To reset the metrics on a node, you have to restart it. The JIRA issue CASSANDRA-8433 requests the ability to reset the metrics via JMX and nodetool.

***Once you’ve gained familiarity with these metrics and what they tell you about your cluster, you should identify key metrics to monitor and even implement automated alerts that indicate your performance goals are not being met. You can accomplish this via DataStax OpsCenter or any JMX-based metrics framework.***


### Analyzing Performance Issues

######################################## (start from here)############################
