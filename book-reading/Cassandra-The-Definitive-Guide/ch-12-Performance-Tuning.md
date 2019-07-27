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











