https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch10.html


Cassandra also features built-in support for Java Management Extensions (JMX), which offers a rich way to monitor your Cassandra nodes and their underlying Java environment. Through JMX, we can see the health of the database and ongoing events, and even interact with it remotely to tune certain values. JMX is an important part of Cassandra.

### Logging
The simplest way to get a picture of what’s happening in your database is to just change the logging level to make the output more verbose. This is great for development and for learning what Cassandra is doing under the hood.

Cassandra uses the Simple Logging Facade for Java (SLF4J) API for logging, with Logback as the implementation.  SLF4J provides a facade over various logging frameworks such as Logback, Log4J, and Java’s built-in logger (java.util.logging)

### Monitoring Cassandra with JMX
In this section, we explore how Cassandra makes use of Java Management Extensions (JMX) to enable remote management of your servers. JMX started as Java Specification Request (JSR) 160 and has been a core part of Java since version 5.0.

