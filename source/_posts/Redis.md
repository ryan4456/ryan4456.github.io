---
layout: an
title: An Investigation About Redis
date: 2020-03-22 17:41:14
tags: "Redis"
---
# Personal Information

Paper: eSystem Design and Development (INFS804)
Student Name: Yanan Li
Student ID:	19058601
Last Updated Date: 20 March 2020

# Abstract

In this paper, five types of data structures in Redis are discussed, which are string, bitmap, hash, list, and set. The communication protocol used by Redis client and server is also explored in detail. Then, the pros and cons are mentioned and there is a comparison among Redis, Memcached, and Aerospike. The best practices are recommended at last.

# Introduction

## Background

In the past two decades, the relational database systems (RDBMSs) have become the predominant solution of storing organizational data in various applications. Codd (1970) proposed that the data in a database is stored in the format of relations. After that, with the development of Structured Query Language (SQL), the manipulation and retrieval of data stored in RDBMSs has been easier. Consequently, RDBMSs are widely adopted in a variety of industries. 
However, with the data handled being larger and larger, RDBMSs may not fully reach the demands of an application. For example, high availability and low latency access to large data sets may be the essential requirements of modern web or mobile applications. In order to meet those demands, corporates could make efforts to simplify their relational schema within RDBMSs, as well as other optimization measures to address RDBMSs’ limitations. However, RDMBSs’ hard disk bottleneck was not addressed; this resulted in the occurrence of NoSQL data models (Nance, Losser, Iype, & Harmon, 2013). 
There are many products of key-value NoSQL databases. Redis ranks the first place with a predominant proportion (DB-Engine, 2020). Although there are lots of research about the usage of Redis in different systems, there is still a  gap in the principle of Redis, and the best practices of Redis.

## Aims

The purpose of this research is to give a systematic overview of the key-value NoSQL database Redis. In this passage, the principle behind Redis will be introduced first. Then the advantages and disadvantages will be presented. After that, there will be a comparison between Redis and other key-value databases. At last, the best practices would give us a clear understanding of when to use Redis and how to use Redis.

# Principle

## Overview

Key-value databases may be one of the simplest and fastest NoSQL datastores. In brief, they can be regarded as a hash table. From the perspective of usage, there is a key to each value stored in the databases. The value could be found and deleted by operating the key. The data structures and communication protocol in Redis will be discussed in Section 4.2 and Section 4.3 respectively.

## Data Structures

As the word suggests, the data structure is the structure of storing data. The complexity of data structures may be quite different. A series of characters may be the simplest data structure and maps may be an example of a complicated data structure. In addition, there are often data structures within data structures, in other words, we can put a map inside a map.
The management of memory and performance are the essential parts in constructing a data structure. The common data structures we often use in different programming languages are tuples, lists, maps, queues, stacks, and so on. When it comes to evaluate the efficiency of the data structure, we actually are evaluating the efficiency of algorithms handling with the data structure. In order to indicate the time efficiency of an algorithm, the big O notation is introduced. From the viewpoint of Redis, we’ll group the data structures according to the notations showed in Table 1.
_Table 1 Big O Notations_

| O Notation	| Description |
|---------------|-------------|
|O(1) |	Time taken on a data structure is a constant. |
|O(N) |	Time taken on a data structure depends on the amount of data contained linearly.|
|O(log(N)) |	Time taken on a data structure has a logarithmic relation with the amount of data contained. |
|O(log(N) + M) |	Time taken on a data structure has a logarithmic and linear relation. Here M is the elements’ number in the sorted set and N is the number of the search range. |
|O(Mlog(N)) |	Time taken on a data structure is logarithmic times linear. Here M is the elements’ number in the target set and N is the number of the search range|

There are many built-in data structures in Redis, which provide a monitory way of organizing data.
The **strings data type** are the basic data types in Redis, and its implementation is contained in `sds.c` (Hack Strings, n.d.). Sds stands for Simple Dynamic Strings. As shown below, we can see the C structure representing a Redis string in the file `sds.h`.
```
struct sdshdr {
    long len;
    long free;
    char buf[];
};
```
The actual string is stored in the character array buf. The attribute len stores the length of buf, which makes retrieve the length of a Redis string an O(1) operation. The field free stores the number of available bytes for use. The two fields, free and len, could be regarded as the metadata of the character array. Any integer, string, image files are stored in character array in Redis and Redis has a mechanism of detecting the type of the data stored in an array.
There are three kinds of operations in Redis for the data structure strings. Table 2, Table 3, and Table 4 shows the setters and getters command, data clean commands, and utility commands respectively.
_Table 2 Setters and Getters Commands_

Command	|Time Complexity	|Description
----------|-------------|---------
GET key|	O(1)	|Get the value of the key. If the key does not exist, return nil. If the value is not a string, return error
SET key value 	|O(1)	|Set the key to hold the string value. If the key already holds a value, it is overwritten, regardless of its type. Any previous time to live associated with the key is discarded on the successful SET operation.
SETNX key value	|O(1)	|Set the value if the key does not exist.
GETSET key value|	O(1)	|Get the old value, then set a new value.
MGET key1 key2	|O(N)	|Gets all the values of the keys listed.
MSET key1 value1 key2 value 2	|O(N), N is the number of keys to set|	Set the keys to values respectively
MSETNX key1 value1 key2 value2|	O(N), N is the number of keys to set|	If the key does not exist, set the keys to values respectively

_Table 3 Data Clean Commands_

Command|	Time Complexity|	Description
------|------------|------
SET [PX\|EX]|	O(1)	|Set the expiry milliseconds or seconds to the key.
SETEX key seconds value	|O(1)	|Atomic operation. Set a value and set the expiry seconds.


_Table 4 Utility Commands_

Command	|Time Complexity|	Description
----------|------------|-------
APPEND	|O(1)	|Appends to a value or set.
STRLEN	|O(1)	|Get the value of the stored string.
SETRANGE	|O(1)	|Replace the string at the given range.
GETRANGE	|O(1)	|Get the string at the given range.

The next one we will talk about is **bitmap data type**. Actually, bitmap is not a data type, it is a set of bit operations on string data type. As string data type is binary-safe and it can store 512MB data, which is 232 bits. There are two types of bit operations: O(1) time complexity bit operations, and operations on groups of bits, which are shown in Table 5 and Table 6 respectively.
_Table 5 Single Bit Operations Commands_

|Command|	Description	|Example|
|------|-----|------|
|SETBIT key offset value|	Sets or clears the bit at offset in the string value stored at key. And the return value is the original bit value stored at offset.|	SETBIT key1 10 1|
|GETBIT key offset	|Returns the bit value at offset in the string value stored at key.	|GETBIT key1 010|


_Table 6 Multiple Bits Operations Commands_

|Command	|Description	|Example|
|-----|------|-------|
|BITOP AND/OR/XOR/NOT |	Performs bit-wise operations between different strings. The provided operations are AND, OR, XOR and NOT. Time complexity is O(N).	|BITOP AND destkey srckey1 srckey2|
|BITCOUNT key [start end]	|Performs population counting, reporting the number of bits set to 1.	|BITCOUNT key1|
|BITPOS	|Finds the first bit having the specified value of 0 or 1.	|BITPOS mykey 0|

The next we are going to talk about is **hash data type**. Due to the data is represented by field-value pairs, it is very convenient to store objects using hash data type. There are no limits in the number of attributes that you could put within a hash. Table 7 shows the commands relating to hash data type.
_Table 7 Hash Operations Commands_

|Command	|Description	|Example	|Time Complexity|
|----|-----|-----|-----|
|HDEL key field [field\…]|	Remove the fields of the key.	|HDEL key1 field1	|O(N) N is removed fields’ number.|
|HEXISTS key field	|Test whether the field exists in the key hash.	|HEXISTS key1 field1	|O(1) |
|HGET key field	|Get the value of the field.	|HGET key1 field1|	O(1)|
|HGETALL key|	Get all fields value	|HGETALL key1	|O(N), N is the number of fields.|
|HKEYS key|	Get all fields|	HKEYS key1|	O(N), N is the number of fields.|

**List data type** is the next one. The terminology List is different in different programming languages. For example, Java LinkedList is a series of elements linked together, while Python List is an array. In general, a List is a sequence of elements and there are two implements to represent it: array or linked list. The list data type in Redis is linked list. No matter how many elements are there in the list, the operation of adding an element at the end or head of the list could be processed in an constant time. However, the disadvantage of this implement is the high time complexity when accessing an element randomly.
The reason why using a linked list to implement Redis list is that the ability to add elements to a quite large list quickly is essential for a database management system.
Table 8 shows some frequently used commands operating list.
_Table 8 Commands Operations on List_

|Command|	Description	|Example	|Time Complexity|
|---|-----|-----|----|
|LPUSH key element [element…]|	Insert elements to the head of the list.	|LPUSH key1 e1 e2	|O(1)|
|RPUSH key element [element…]|	Insert elements to the tail of the list.	|RPUSH key1 e1 e2|	O(1)|
|LPOP key	|Delete and return the first element of the list.	|LPOP key1	|O(1)|
|RPOP key	|Delete and return the last element of the list.|	RPOP key1	|O(1)|

The last data structure introduced is Redis **set**. Redis set is random collection of strings. The attractive aspect of Redis set is that the time complexity of operations on adding, deleting, and checking the existence of an element is constant time. Table 9 is the commands operating set.
_Table 9 Commands on Set_

|Command	|Description	|Example	|Time Complexity|
|----|-----|-----|----|
|SADD	|Add elements to the set.|	SADD set1 e1 e2	|O(N), N is the number of elements to be added.|
|SREM	|Delete and return elements from the set.	|SREM set1 e1 e2	|O(N), N is the number of elements to be removed.|
|SCARD	|Get the number of elements in the set.	|SCARD set1|	O(1)|
|SDIFF|	Get elements which equals the first set subtracting the following sets.	|SDIFF set1 set2	|O(N), N is the number of elements in all sets.|
|SISMEMBER|	Check if the elements exists|	SISMEMBER set1 e1	|O(1)|

## Communication Protocol

The model of Redis working on is client-server. Just like other client-server models, the server and client service needs to use a protocol to communicate with each other. Communication protocol is some fixed agreement or standards that clients and servers follow in order to exchange messages successfully. 
The Redis protocol contains two parts, which are a header and a body. The body of a request is made of command and arguments, while the body of a response is the data payload. The header of a response contains the information, such as the success or failure flag of the request. 
Let’s see the protocol in detail from the perspective of a request and response.
As it is shown in Figure 1, there are two parts in any Redis client request. The first part is header, which contains information about the number of arguments of the command. The second part is the body part, which contains three small parts. The first part of a body is the number of bytes for every argument, and the second part of a body is the actual arguments, and the last part of a body is carriage return and line feeds.
![Figure 1 Request Protocol Structure](/uploads/redis1.png "Request Protocol Structure")
Considering the response in Redis, there are two types. The first type is the response of the commands that do not expect a return value. A plus sign or minus sign would be contained in the response. The plus sign indicates that the request was successful and the minus sign represents that the request failed. The second type of response is the response of the commands that retrieve data from the server. ‘$-1’ will be the response if an error happens and the response would contain a dollar sign, the size of the response, and the retrieved data if the request succeeds.
After having a basic knowledge of data structures provided by Redis and the communication protocol between Redis client and server, the advantages and disadvantages of Redis will be discussed in the next section.

# Discussion

## Pros and Cons

There are two benefits of using Redis as a key-value database, which is the usage of scripting in the server-side and the abundant supporting infrastructure around Redis.
One of the advantages of using Redis is server-side scripting. According to Zhang, Tudor, Chen, and Ooi (2014), server-side scripting is supported by Redis and they pointed out that Redis server could directly handle some processing using Lua scripts. They called this procedure “Redis server-side data analytics”.
Another advantage of Redis is the ecosystem of tools around it. Just as Anthony and Rao (n.d.) stated, there is great user-friendliness with Redis, which is the result of the appearance of its capabilities of monitoring and various built-in commands. They also introduced that Redis has not only a number of users but also a great policy of documentation, which reduces people’s workload. Both the installation and configuration is pretty easy and the default configuration may be the most suitable configuration for the majority of users. Different use scenarios with sample codes are also available in official documents. In considering different languages, there are various adaptable libraries provided. All the components within the ecosystem make us use Redis efficiently without any difficulties.
Although there are many advantages, there are also disadvantages of using Redis.
The first drawback is the memory fragmentation problem. Liu and Yuan (2019) explained that the allocation of memory in Linux should begin with an address which is defined by the paging scheme of the memory management unit. As a result, the allocated memory can only be inflexible size pages leading to the extra memory created. This extra memory space is internal memory fragmentation. They also pointed out the reason why Redis has the issue of memory fragmentation. The default memory management component in the most recent version of Redis is Jemalloc. In order to reduce the outside fragmentation, Jemalloc classifies the request of memory distribution. Despite that, this design pattern has not made any optimization for the internal fragmentation issue, and the common use case of Redis is cache. For the purpose of processing the expired data with various length and writing these data, there are significant high frequencies of applying and clearing memory in Redis. Consequently, there are obvious increases in occupied memory and fragmentation ratio.
The second problem of using Redis is scalability. Kago and YamashiTa (2014) discussed that the pattern that Redis works is a singular process and there is no support for the cluster system. What’s more, Redis provides a basic master-slave reproduction function without a failover system. This means that there is no horizontal scalability with Redis. 

## Comparison with Other Key-Value Databases

There are many other key-value database products, such as Amazon DynamoDB, Memcached, Aerospike, and so on. In this section, the comparison among Redis, Memcached, and Aerospike will be discussed. The comparison will include the analysis of throughput, latency which is conducted by Anthony and Rao (n.d.).
According to Anthony and Rao (n.d.), when it comes to read-heavy and balanced workload situations, Memcached has the best performance concerning throughput and Aerospike and Redis has nearly the same efficiency. Meanwhile, in write-heavy conditions, the medium of Redis and Aerospike’s capacity is more than that of Memcached, and Aerospike has better performance than Redis. They explained that is the result of different data storage mechanism in these three databases.

About latency, Memcached has much less read latency than Redis and Aerospike when dealing with read-heavy and balanced burden, and Redis has a lightly higher latency than Aerospike. They observed that the minimum read latency for Redis, Memcached, and Aerospike is 195.4 μs, 170 μs, and 180 μs respectively, and they concluded from this result that there is significant extra workloads for the server to deal with requests from a large number of clients. They also summarised that the design of Redis’s single thread results in an inefficient performance compared to the multiplied thread pattern of Memcached.

## Best Practices

As it is discussed in the previous several sections, there are various data structures in Redis. These data structures could be applied in different use cases.
As for the list data structure in Redis, there are two typical situations where Redis could be used. The first one is collecting the newest posts published by users in social networks. For example, the latest posts in the well-known social network Twitter are stored in Redis lists. In order to illustrate a common social network case stage by stage, imagine the scenario that your personal main page shows the latest blog list and you want to accelerate the speed of displaying the list. This could be accomplished by the following two steps: when a blog is published, add the blog identity into a list using the command LPUSH; Use the command “LRANGE 0 9” to get the newest ten posted blogs. The second use case is communication between procedures. Just like the famous consumer-producer pattern, items are pushed into a list by producers and consumers fetch items from the list to handle.
The common use cases relating to bitmaps are real-time analysis and saving bool information that requires high performance with efficient space. For instance, the administrator wants to know who is the user that has the longest daily visit of the website. He or she counts days beginning with zero, which represents the day that the website was online, and every time the user opens the website he or she set the bit with the command SETBIT. By using this method, there is a small string for every user recording their daily visits. As last, it is possible to figure out how many days the user has visited the website with the command BITCOUNT.

# Conclusion

In this paper, many aspects of Redis are discussed. First, there are five types of data structures introduced, which are string, bitmap, hash, list, and set. Then, the communication protocol is explained in detail. After that, the advantages such as server-side scripting, and sustainable ecosystem, and disadvantages such as the fragmentation of memory, and the complexity scalability are mentioned. Then, the comparison among Redis, Memcached, and Aerospike is conducted. Finally, the best practices are described. For example, there are two typical use cases of list data structure and two use scenarios of using bitmaps provided.
This paper provides the basic concepts of Redis for future studies. It can help researchers to apply more advanced studies about Redis.
However, this paper has some limitations. The primary restriction for this study is the limited time. The lack of peer-reviewed resources is another limitation.

# Reference 

<pre>
Anthony, A., & Rao, Y. N. M. Memcached, Redis, and Aerospike Key-Value 
    Stores Empirical Comparison.
Codd, E. F. (1970). A Relational Model of Data for Large Shared Data Banks. 
    <i>Commun. ACM, 13</i>(6), 377–387. https://doi.org/10.1145/362384.362685
DB-Engine. (2020). DB-Engines Ranking of Key-value Stores. Retrieved March 21, 
    2020, from https://db-engines.com/en/ranking/key-value+store
Hacking Strings. (n.d.). Retrieved March 21, 2020, from https://redis.io
    /topics/internals-sds
Kago, M., & Yamashita, A. (2014). Development of a scalable and flexible data 
    logging system using NOSQL databases. <i>Proceedings, 14th International 
    Conference on Accelerator & Large Experimental Physics Control Systems</i>, 
    532. http://www.jacow.org
Liu, Q., & Yuan, H. (2019). A High Performance Memory Key-Value Database 
    Based on Redis. <i>JCP, 14</i>(3), 170-183.
Moniruzzaman, A. B. M., & Hossain, S. A. (2013). NoSQL Database: New Era of 
    Databases for Big data Analytics-Classification, Characteristics and Comparison. 
    <i>International Journal of Database Theory and Application, 6</i>(4).
Nance, C., Losser, T., Iype, R., & Harmon, G. (2013). Nosql vs rdbms-why there is 
    room for both.
Zhang, H., Tudor, B. M., Chen, G., & Ooi, B. C. (2014). Efficient in-memory data 
    management: An analysis. <i>Proceedings of the VLDB Endowment, 7</i>(10), 833-836. 
    https://doi.org/10.14778/2732951.2732956
</pre>