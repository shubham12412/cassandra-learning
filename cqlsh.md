### Describing the Environment in cqlsh

1) DESCRIBE CLUSTER;
2) DESCRIBE KEYSPACES;
3) SHOW VERSION;

### Creating a Keyspace and Table in cqlsh

4) CREATE KEYSPACE my_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3} ;
5) DESCRIBE KEYSPACE my_keyspace;
6) USE my_keyspace;
7) CREATE TABLE user ( first_name text , last_name text, PRIMARY KEY (first_name)) ;
8) DESCRIBE TABLE user;


### Writing and Reading Data in cqlsh

9) INSERT INTO user (first_name, last_name ) VALUES ('Bill', 'Nguyen');
10) SELECT COUNT (*) FROM user;
11) SELECT * FROM user WHERE first_name='Bill';
12) DELETE last_name FROM USER WHERE first_name='Bill';
13) DELETE FROM USER WHERE first_name='Bill';
14) TRUNCATE user;
15) DROP TABLE user;

--------------------------------------------------------------------------------------------------------------------

16) ALTER TABLE user ADD title text;

17) SELECT first_name, last_name, writetime(last_name) FROM user;
18) 


------------------------------------------------------------------------------------------------------------------

https://thelastpickle.com/blog/2016/02/18/dropping-columns.html

