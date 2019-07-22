First let’s take a look into some other attributes of a column that we haven’t discussed yet: ***timestamps*** and ***time to live***. These attributes are key to understanding how Cassandra uses time to keep data current.

### TIMESTAMPS
Each time you write data into Cassandra, a timestamp is generated for each column value that is updated. Internally, Cassandra uses these timestamps for resolving any conflicting changes that are made to the same value. Generally, the last timestamp wins.

### TIME TO LIVE (TTL)
One very powerful feature that Cassandra provides is the ability to expire data that is no longer needed. This expiration is very flexible and works at the level of individual column values. The time to live (or TTL) is a value that Cassandra stores for each column value to indicate how long to keep the value.

----------------------------------------------------------------------------------------------------------------------

Collection types are very useful in cases where we need to store a variable number of elements within a single column.

For optimal read performance, denormalized table designs or materialized views are generally preferred to using secondary indexes.





