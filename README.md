# cassandra-101
cassandra 101 basic tutorial notes


Cassandra no sql database manage large volume fast changing data

Lack of joins

Cqslh – Casandra

Releational database like mysql an dpostgre sql – ensure the read and write are consistent, so that all users see the latest correct version of data. Add overhead. 


High volume and high velocity – no sql databases– non-relational databases. No use sql for defining and manipulating data.  Casandra is wide-column no sql. Use table as data structure.
Primary keys used to access data. Not enough find rows. Cassandra run clusters of servers. No single server. Partition key defines which node in the cluster to use when storing or retirieving a row. A clustering column defines the order in which rows are stored.

Servers in a Cassandra cluster are called nodes. That convention is followed in this course.

Cassandra has query language called CQL – Cassandra query language. Similar to sql but has more restrictions. 

CQL has for define data structures – CREATE TABLE, CREATE TRIGGER, CREATE INDEX, DROP INDEX, UPDATE

Cassandra does not have a fixed schema. Some rows may have different columns than other rows.

  
  ![image](https://user-images.githubusercontent.com/25869911/137604887-2cee41c0-17be-4abf-8d7c-e4d96c655aeb.png)

  
  
  
  
 
Cassandra eventyally consistent databases – replicas of a row may have different versions of the data for a short time.

Cassandra keeps multiple copies of data of different nodes(servers), in case a node fails, users can get copies of data on  replicas on different nodes.

RDBMS  different from Cassandra

In Cassandra Top label of data structures is keyspace. This is like a schema in releational databases. Keyspaces are logical containers for tables, indexes and other data structures.

In addition providing namespace for organizing data, the keyspace has attributes that control some important features of Cassandra.

When you define keyspace, also you define a replication strategy. Replica(high availability features)

2 kinds of replication strategies
-	Simple replication – sets a number of replicas when using a single data center for a cluster
-	Network topology replication – sets a number of replicas and manages replicas across multiple data centers


CREATE KEYSPACE
	perfmonitor
WITH
	Replication = {‘class’:’SimpleStrategy’,
    			‘replication_factor’: 3};


When we have our keyspace, we can creat a Table with command CREATE TABLE

Attributes – Application name, Process ID, Host ID, Operation System priority, CPU time, Number of IO operations

CREATE TABLE app_instance (
	Id uuid,
	App_name text,
	Proc_id text,
	Host_id text,
	Os_priority int,
	Cpu_time int,
	Num_io_ops int,
	PRIMARY KEY (id))


Columns can be single values or group of multiple values(called collections)
-	SINGLE - Int, Text, Float, Timestamp, Date, Blob(byte stream), UUID
-	Collection – List, Map, Set

In table attribute
 	Os_priority map<text,int>     -> average 2, minimum 1, maximum 3

Data modeling – selecting primary key. Carefully choice primary key

Host_id and prc_id are unique to identify the app

CREATE TABLE app_instance (
	App_name text,
	Proc_id text,
	Host_id text,
	Os_priority int,
	Cpu_time int,
	Num_io_ops int,
	PRIMARY KEY (host_id,proc_id))

Primary key host id is, first attribute in primary key is used as the partition key, which determines which node(server) the row is stored on. Rest of the primary key are used as clustering key, which determines how data is ordered on the disk 

SELECT * FROM app_instance WHERE host_id= ‘AppServer1’ AND proc_id=’33833’

Change the default sort order of rows done when create table

CREATE TABLE app_instance (
	App_name text,
	Proc_id text,
	Host_id text,
	Os_priority int,
	Cpu_time int,
	Num_io_ops int,
	PRIMARY KEY (host_id,proc_id))
WITH CLUSTERING ORDER BY (proc_id DESC)

Cassandra doesn’t provide to sort query results at query time, so we have to consider sort order when creating a table

![image](https://user-images.githubusercontent.com/25869911/137604895-ddf37667-3658-4f5c-8b40-22f395a0643e.png)

 

Knowing primary key to loop up faster the data

SELECT * FROM App_instance WHERE app_name = ‘WebStoreFront’.    – this is bad example quering with something is not primary key, unprecdictable result

You can create a index, using secondary index, they created table that updated along with the base table on which the index is defined. Both tables need to be updated when data is inserted
This can lead to additional overhead. Important not to use secondary indexes on columns with very low or very high numbers of unique values

CREATE INDEX
	Appname_idx
ON 
	App_instance(app_name)


 Data model -> partition key and clustering key.

Write data to cluster :

There is no distinction of master and worker nodes in Cassandra. All nodes run the same services.

Consistent hash fucntions - A partitioner is a hash function that maps partion key to a node(server) in the cluster

When data is written to a node. Replicas are created depend replicas strategies.

Simple replication -> replicate to the next node of the ring(small clusters)
For large clusters, that use multiple data centers, with network topology strategy to take into account the rack and the data center locations of various nodes. This improving faoult tolerance and availability of the cluster.


Start Cassandra
cassandra -f

check Cassandra status
service Cassandra status

para entrar a Cassandra command
cqlsh

Create keyspace is like schema

CREATE KEYSPACE perfmonitor
WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};


See what command used perfmonitor keyspace
Describe perfmonitor

-	Select that we are using this keyspace
USE perfmonitor

-	Create table
CREATE TABLE app_instance (
            app_id int,
    app_name varchar,
    proc_id varchar,
    host_id varchar,
    os_priority int,
    cpu_time int,
    num_io_ops int,
PRIMARY KEY (host_id, proc_id)
)
WITH CLUSTERING ORDER BY (proc_id DESC)

-	To see the app instance table how was create
DESCRIBE app_instance

-	Clear screen
Clear

-	Insert values to the tables
insert into app_instance 
    (app_id, host_id, proc_id, app_name,os_priority,cpu_time,num_io_ops) 
values 
   (1,'Host1','Proc1','App1',90,145,250);

insert into app_instance 
    (app_id, host_id, proc_id, app_name,os_priority,cpu_time,num_io_ops) 
values 
   (2,'Host2','Proc2,'App2',60,155,550);


-	Query the table
Select * from app_instance;

Select * from app_instance where proc_id = ‘App4’;

Query by where app_name -> will be throw error, because
App_name is not primary key

-	Create index(secondary indexes)
Create index appname_idx on app_instance(app_name);

Describe appname_idx;

-	Now this going to working
select * from app_instance where app_name ='App1';


cassnadra is often used to store large volumes of data.

Numeric data – int(32 bit signed integers (2,147,483,647 – 2147,483,647)), bigint(64 bit signed integers (9,223,372,036,854,775,808  to – 9,223,372,036,854,775,808)), tinyint(1 byte numbers (signed -127 a 127 , unsigned 0 - 255)), varint(store arbitrary precision integers with variable length encoding), decimal(decimals define variable precision decimal values, a column with 10 decimals, two of which are used for decimal values, support 12345678.01), double(store 64 bit IEEE 754 FLOATING POINT NUMBERs), float(store 32 bit IEEE 754 floating point number)


Sign – one bit is used to indicate a positive or negative number. 31 bits used to represent the number

3 types of data types in Cassandra – ASCII(127 characters, character strings), Varchar(UTF-8 enconded strings, 128,000 characters) and text(store UTF-8 enconded strings, alias for VARCHAR)

 2 TIME DATA TYPES – timestamp(date plus time, encoded as 8 bytes since the unix epoch, which is defined as starting at midnight on January 1, 1970. The number of milliseconds since the epoch, you can use number of milliseconds since the epoch or ISO 8601 FORMAT) and timeUUID (Universally unique identifier based on time and the MAC address of the device generation the identifier and the time the identifier is generated, includes time and enables sorting on time)

UUID –random strings based on the mac address. 32 charter strings

Is 8601 format 
	yyyy-mm-dd HH:MM
             yyyy-mm-dd HH:mm:ss
             yyyy-mm-dd HH:mmZ
            yyyy-mm-dd HH:mm:ssZ

z is time zone – four digits based on RFC 822 standard


Collections used to sotre multiple atomic values in a single column

Lists – order-preserving collection that may contain duplicates
Sets – order-free collections without duplicate.  (more recommended to use)
Maps – sets of key-value pairs- name and value, 

Cassandra can store up to 64000 items in a collection, best practice keep the collection small.

Cassandra provides a tuples data type to create structured collections

Tuples – are ordered lists of attributes. Fixed structures. Different from lists and sets, which can have varying lengths.  Attributes in tuples can be of different types.

Location<tuple<decimal,decimal,int>>  -> <latitude, logitud, altituded
Order matter

 Tuples are used when you want to logically group several attributes and you will not need to add any new attributes in the future.


Queries drive data model design in cassandra

Data modeling in releation databases – entitites and relationships

	Entities – physical servers, virtual machines, applications, processes

	Have releation ships
 
 ![image](https://user-images.githubusercontent.com/25869911/137604899-802a3b85-ab0e-4b71-8585-bb7736712cf2.png)


When we desing data models in Casandra, we don’t start with entities. We start with the queries we want to run.
what is that we want to report on ? we are concerned about application performance, so we might want a report on servers that are running with high CPU utilization

in a normalized model, we would have a table for servers and a table for server metrics.

In Cassandra , we don’t necessary need multiple tables, instead we are using single table.
Servers by CPU utilization. This table contains server name, operation system name, operationg system version, memory size, disk storage, ssd storage, host name and ip address. But also have data about performance metric at different points of time.  Timestamp, cpu utilization, number of processes, IO read in last minute, IO writes in last minute and free memory.

In relational mode :
(Server) has 1 to many relationship to (server metrics)


Cassandra - Fast read and fast right in hugh volume of data
No joins and duplication of data

Relational databases design to reduce the chance of intrudcing data anomalies, like reporting an outdated address for a customer.

Cassandra trade higher performance with big data for using extra storage space and risking some data anomalies.

Write and read data quickly adavantage of Cassandra

If you use storing large number of data points use tynyint, not use bigint.

Optimization in Cassandra -instead of storing an entire row each time a new set of server metrics come in, we can store those metrics in a new column in an existing row. Use wide table instead narrow table.

 ![image](https://user-images.githubusercontent.com/25869911/137604901-da5490bd-598b-4001-b25f-84b92b2e4b1c.png)


 
We use denormalizing instead of joining and sorting in Cassandra

Denormalizing is done by 2 ways:

1 - multiple copies of data – keeping redundant copies of data across rows and tables

2 – nonatomic columns – using collections to store multiple values instead of using a separate table.

For different sort of the element- we need to created different tables


Duplication avoids joins

serverName, averagedis utilization, operation system name, operation system versions – you know these have not huge changes.


 Time series data -  simple way to model time series in Cassandra is to use what is called a wide row.

Normal :

CREATE TABLE server_cpi_utilization (
Server id text,
Measure_time timestamp,
Cpu_utilization int,
PRIMARY KEY (server_id, measure_time))


Store so much data in single partition of data can lead to poor query performance

Fine-grained partition – we could partition by server and by date(compound key) :

CREATE TABLE server_cpi_utilization (
Server id text,
Measure_time timestamp,
Cpu_utilization int,
PRIMARY KEY (server_id, measure_date), measure_datetime)

Another is Time to Live(TTL) example, ater ttL the data is marked for deletion

Using normal table, 
When insert data

INSERT INTO server_cpi_utilization (server_id, measure_time, cpu_utilization) VALUES (‘AppServer1’M ‘2017-05-20 07:01:00’, 83) USING TTL 86400;

86400 Is second – equivalent 1 day

Another is descending sort order – descending time sort order, newest first, oldest last

CREATE TABLE server_cpi_utilization (
Server id text,
Measure_time timestamp,
Cpu_utilization int,
PRIMARY KEY (server_id, measure_time))
WITH CLUSTERING ORDER BY (measure_datetime DESC)


LAB : 

Use perfmonitor;
 
create table server_cpu_utilization (
               server_id varchar,
               meaure_time timestamp,
               cpu_utilization int,
               primary key (server_id, measure_time));

insert into server_cpu_utilization (server_id, measure_time, cpu_utilization) values ('appserver1','2017-05-20 07:01:00', 83) using ttl 86400;
 
insert into server_cpu_utilization (server_id, measure_time, cpu_utilization) values ('appserver1','2017-05-20 07:02:00', 85) using ttl 86400;
 
insert into server_cpu_utilization (server_id, measure_time, cpu_utilization) values ('appserver1','2017-05-20 07:03:00', 87) using ttl 86400;
 

-	To know how many time left to live
Select cpu_utilization, TTL(cpu_utilization) from server_cpu_utilization;


Cassandra we design table to answer specific queries


Satisfy multiple queries with 1 table

CREATE TABLE devices (
Id uuid,
Device_name text,
Ip_address set<text>,
Location map<text,text>,
Installation_Date date,
Installation _year int,
PRIMARY KEY ((installation_year), installation_Date, id))

Lets add manufacturer -> 

CREATE TABLE devices (
Id uuid,
Device_name text,
Ip_address set<text>,
Location map<text,text>,
Installation_Date date,
Installation _year int,
Manufacturer text,
PRIMARY KEY ((installation_year), installation_Date, id))

 Installation year as a partition key -> determines which node stores the data.
Installation date and id -> clustering key. We can look up rows in the table using the primary key.

We want to select by manufacturer name
CREATE TABLE devices (
Id uuid,
Device_name text,
Ip_address set<text>,
Location map<text,text>,
Installation_Date date,
Installation _year int,
Manufacturer text,
PRIMARY KEY ((installation_year), manufacturer, id))

We have 2 tables, tuplicate data, data duplication is common practice.

Another options :  secondary index.
CREATE INDEX device_manufacturer_idx ON devices(manufacturer)
This enables :
SELECT * FROM devices WHERE manufacturer = ‘Acme Server’

When Secondary indexes are useful 
-	there are many rows that have the indexed value
-	tables do not have a counter, an autoincrement feature of Cassandra.
-	The column is not frequently updated


High cardinality index

When you have few rows with a particular indexed value. Indexes do not perform well in those cases and should be avoided.

Server serial number is associated with only a single machine, so there will be many distinct serial numbers. Secondary indexes do not work well on a column with serial numbers. In this cases create another table with appropriate primary key

CREATE TABLE devices_by_serial_number ( 
Id uuid,
Device_name text,
IP_address set<text>,
Location map<text,text>,
Installation_Date date,
Installation _year int,
Manufacturer text,
Serial_number
PRIMARY KEY ((installation_year), serial_number, id))


Materialized views

You could manage multiple tables when you need to denormalize. Each time you add a row to one table, you would add it to the others. You could also implement operations to keep the data synchronized. The easier ways is materialized views.

Materialized view is a table that is managed by Cassandra, Cassandra will keep data in-sync between tables and materialized views based on those tables

CREATE TABLE devices ( 
Id uuid,
Device_name text,
IP_address set<text>,
Location map<text,text>,
Installation_Date date,
Installation _year int,
Manufacturer text,
Serial_number
PRIMARY KEY (id))

Create materialized vie on this table

CREATE MATERIALIZED VIEW devices_by_Serial_number 
AS
SELECT
Serial_number, device_name, manufacturer
FROM
Devices
WHERE
Serial_number IS NOT NULL AND id IS NOT NULL
PRIMARY KEY(serial_number,id);

Materialize dviews can help reduce the overhead of managing denormalized tables

Limitations :
-	The primary key of the base table must be in the primary key of the materialized view
-	Only one column can be added to the materialized view primary key.

Add extra steps when update operation and data replication

General rule of thumb is to expect about a 10% performance

Materialized view overhead : write = +10% read = +0%
Trade off




Uuid and delete exercise

CREATE TABLE devices (
    id uuid,                         
    device_name text,
    ip_address  set<text>,
    location map<text, text>,
    installation_date date,
    installation_year int,
    manufacturer text,
    serial_number text,
    PRIMARY KEY (id));


-- Insert example rows using the UUID function to create unique IDs
Insert into devices  
   (id, device_name, ip_address, location, installation_date, installation_year, manufacturer, serial_number) 
Values 
(uuid(), 'Server1', {'192.168.0.1'}, {'data center':'DC1', 'rack':'Rack1'}, '2015-01-20', 2015, 'Acme', 'SN12345');


-- Delete a column from a row vs. deleting an entire row.

—- Note, the following commands do not have UUIDs specified. They will 
—- be different for each row inserted. To view UUIDs for each row, enter the 
—- following command:

select id, device_name from devices;

-	Delete column

delete device_name from devices where id =  <insert a UUID from one of the rows inserted above>

-	Delete row
delete from devices where id = <insert a UUID from one of the rows inserted above>


estimating data size 

column data – metadata associated with columns +column values
-	Need column name : Cassandra stores column names with data
-	Column value : size is determined by the data type
-	overhead value : it is usually 15 bytes, but could be as many as 23 if a TTL is used
o	text or varchar size, little more difficult to estimated
	average(length of text or varchar)+1 byte
row data – metadata associated with each row
-	sum(stored column sizes) + 23(overhead) bytes
indexes – secondary index data

inputs to estimating table size
1 - determine column size based on data type(column size)
2 – estimate the percentage of rows that will have this column(column use percentage)
3 – estimate the number of rows in the table(row count)
4 – expected column storage = column size * column use percentage
5 – expected row storage is the sum of all columns in expected column storage.(add 23 bytes for overhead)
6 – table size = expected row storage * row count

Factor in the index size

1 each table has to keep an index of the primary key
2 the size of the columns in the primary key determines the size of the index
3 each index entry required 32 bytes of overhead

Total estimated table size =  Table data size + index size


Cassandra replication – high availability and query performance
Cassandra cluster organized in a ring.
5 node cluster
Node 1-> node 2 -> node 3 -> node 4 -> node 5 -> node 1

Keyspace – define number of replica , 1 -> one copy

Local setting -replication 1 is reasonable, production -2 or more replication

Simple strategy – local development, single rack and data center, clockwise next node
Network toplogy strategy – multiple data center and racks. 4 , large, high availability.





Consistency levels 
Data replication – we create the possibility that copies of data may get out of sync.

High consistency – required all replicates need to be sync.

Any(consistency level) – high availability and low consistency. 
Two – tow replicas return the same value
Three – three replicas return the same value
Quorum – a majority of replicas return the same value
All – high consistency and low availability

Specify consistency at three levels
Entire cluster – consistency set by default for entire clusters
Single DC – CONSISTENCY SET WITHIN A SINGLE DATA CENTER
Single IO -most fine-grained option; allows for override of default cluster and data center values



Casssandra architecture – 

How Cassandra execute the queries

Hash function to where to read and write, coordinator node decide

Cassandra use timestamp to know from various replicas what data is the most updated data.in the consistency level rule applied. Coordinator use read repair to update the outdated not sync data in the replica.

Cassandra use optimized write process data structures. Combines feature for data integraty, high availability and low latency. Cassandra write data in 3 different data structures.
The commit log, memtables and ss tables. Combinantions of these 3 help Casandra achieve high availability , fast write and low latency read capabilities.

Commit log -append only log, every write operations is written to the commit log on persistent storage. The commit log enables Cassandra to recover writes,even if power fails or  a server crashes before writing data to the table data structure. Order is received. Data is only read commit log during the recovery operation, fast writes. 

Memtables – are write back caches of data partitions. Is in memory. The results in low latency reads and writes. If have power failure, memtables loose data but we have commit log has copy of same data of memetables. Rows in a memtable are stored in sorted order.

Sstables – when memtables reach a configured threshold, the data is flushed or written to disc. Persistent storage(disc or ssd). Stored as sorted string table. Ss table are inmutable. Once written, data is not changed. Number of files created – data tables, indexes, statics(meta data), compression information and bloom filters(is a data structure use to optimize read operations).


Bloom filters – when client query, node n nned to now what sst table is , bloom filter to narrow down sstable, test is entity is element of set, sometimes false positive wrong answer returned.
Little space and fast, only generate false positives, tunable error rate.

Deletes and tombstones – cansandra delete differente from releational database delete.
We consider how our data will change, both in terms of new data  being added and existing data being deleted. Relational database delete, and the space is available. But in Cassandra, sst table is immutable, the data when is memtable is mutable(data can be literally deleted in memtable), instead perform delete, we execute special kind of write operation that creates a marker indicating the data has been deleted.this data structure is called a tombstone. When Cassandra fetches data for rows, if fetches tombstones as well. Tomstones are write operations, it is possible that a write operation fails. A node can fail during a write operation and the tombstone is not written to the sstable. If this happens, we could have a ghost or zombie data that should have been marked as deleted but is still aroud. If node crash down time is short, it will get a message from another node to delete specific data. Down time of the node need to be shorted than specified in a grace period parameter. Other nodes will send delete messages during the grace period but will stop once the grace period is exceeded. In this case, data that should have been deleted is still in place.  Cassandra attempt to replay missed updates including tomstones, when a node comes back online. Data base admin can also run a node repair tool to crrect for inconsistencies between replicas. Data actually removed in database in the process named compaction.



 ![image](https://user-images.githubusercontent.com/25869911/137604913-a49f85e9-070c-4ecd-b02d-07263ef522a8.png)




Compaction is the process of merging data from sstable to new sstable, to keep down number of sstables. This help reclaim spaces that has been marked with tombstones. After compacting, old sstable continues to exist until java virtual machine perfurms a garbage collection operation on the Cassandra node.
Problems of the compactions – some time the new and old sst table, both table will continue to exist for some time

2 ways to do compactions :
-	leveled compaction
o	your application needs low-latency read operations
o	there tends to be more read than writes
o	rows are frequently updated
-	size-tiered compaction
o	when disk I/O should be minimized
o	there are many write operations by your application
o	rows are written once, such as in IoT application

5 BEST PRACTICES IN CASSANDRA

-	model based on queries
o	focus first on queries and design tables to support those queries(your model is driven by queries)
o	use multiple tables as needed (small number of queries)
-	denormalize
o	is a common practice in no sql databases
o	helps to compensate for not having joins
o	improves query performance
-	plan for sort order
o	CQL SELECT statements do not support an ORDER BY CLAUSE.
o	ORDER BY is specified when a table is created, not when a query is executed.
o	Cassandra 3 – materialized views can be used in Cassandra 3 to maintain tables with common data but different sort orders
-	Understand replication strategies
o	Replicating data enables high availability
o	Simplestrategy is useful for local development instances and small Cassandra clusters
o	Network toplogy strategy is used when designing for multi-rack and multi-data center high-availability clusters.
-	Understand eventual consistency
o	Using replicas introduces opportunities for copies of data to be out of sync or inconsistent.
o	Read and write quorums are set according to your need for low-latency read and writes as weel as your need for consistent data.

