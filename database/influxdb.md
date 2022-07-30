# influxdb学习
influxdb是一种基于列式存储的时序数据库

## 1.8.x
  InfluxDB专门针对时序数据做了优化。数据来源通常是分布式监控（传感器）, 巨型网站的点击数据,或者各种金融报表。influxdb针对插入和查询效率做了优化，相应地削弱了修改和删除的效率。基于influxdb的这种特性，有以下几种方法可以一定程度上提高修改和删除的效率
  - 如果要更新一个节点，可以选择插入一条 measurement、tag set和timestamp都完全相同的节点.
  - 对于删除操作。可以选择删除一个序列，而不是基于字段的值做删除。例如，可以先查询字段取回时间戳，然后基于时间戳做删除操作 
  - tag暂时不可修改，如果要修改tag只能删掉旧数据重新插入
  - tag不可移除 #8604
### Terminology(术语)
```
The table below is a (very) simple example of a table called foodships in an SQL database with the unindexed column #_foodships and the indexed columns park_id, planet, and time.
+---------+---------+---------------------+--------------+
| park_id | planet  | time                | #_foodships  |
+---------+---------+---------------------+--------------+
|       1 | Earth   | 1429185600000000000 |            0 |
|       1 | Earth   | 1429185601000000000 |            3 |
|       1 | Earth   | 1429185602000000000 |           15 |
|       1 | Earth   | 1429185603000000000 |           15 |
|       2 | Saturn  | 1429185600000000000 |            5 |
|       2 | Saturn  | 1429185601000000000 |            9 |
|       2 | Saturn  | 1429185602000000000 |           10 |
|       2 | Saturn  | 1429185603000000000 |           14 |
|       3 | Jupiter | 1429185600000000000 |           20 |
|       3 | Jupiter | 1429185601000000000 |           21 |
|       3 | Jupiter | 1429185602000000000 |           21 |
|       3 | Jupiter | 1429185603000000000 |           20 |
|       4 | Saturn  | 1429185600000000000 |            5 |
|       4 | Saturn  | 1429185601000000000 |            5 |
|       4 | Saturn  | 1429185602000000000 |            6 |
|       4 | Saturn  | 1429185603000000000 |            5 |
+---------+---------+---------------------+--------------+
Those same data look like this in InfluxDB:

name: foodships
tags: park_id=1, planet=Earth
time			               #_foodships
----			               ------------
2015-04-16T12:00:00Z	 0
2015-04-16T12:00:01Z	 3
2015-04-16T12:00:02Z	 15
2015-04-16T12:00:03Z	 15

name: foodships
tags: park_id=2, planet=Saturn
time			               #_foodships
----			               ------------
2015-04-16T12:00:00Z	 5
2015-04-16T12:00:01Z	 9
2015-04-16T12:00:02Z	 10
2015-04-16T12:00:03Z	 14

name: foodships
tags: park_id=3, planet=Jupiter
time			               #_foodships
----			               ------------
2015-04-16T12:00:00Z	 20
2015-04-16T12:00:01Z	 21
2015-04-16T12:00:02Z	 21
2015-04-16T12:00:03Z	 20

name: foodships
tags: park_id=4, planet=Saturn
time			               #_foodships
----			               ------------
2015-04-16T12:00:00Z	 5
2015-04-16T12:00:01Z	 5
2015-04-16T12:00:02Z	 6
2015-04-16T12:00:03Z	 5
InfluxDB tags ( park_id and planet) are like indexed columns in an SQL database.
InfluxDB fields (#_foodships) are like unindexed columns in an SQL database.
InfluxDB points (for example, 2015-04-16T12:00:00Z 5) are similar to SQL rows.
```

### key concepts(核心概念)
1. database
- A logical container for users, retention policies, continuous queries, and time series data.
- freetsdb（基于influxdb-1.8.x改造的，拥有集群的能力）在初始化完成后是没有“数据库”存在的,而influxdb 1.x会内置一个名为“_internal”的系统库

2. field(字段)
- The key-value pair in an InfluxDB data structure that records metadata and the actual data value. **Fields are required in InfluxDB data structures and they are not indexed** - queries on field values scan all points that match the specified time range and, as a result, are not performant relative to tags.
- 字段由 key-value组成（field key,field value）。字段组合（field set）是一组k-v的集合

3. measurement
The part of the InfluxDB data structure that describes the data stored in the associated fields. Measurements are strings.

4. point
- In InfluxDB, a point represents a single data record, similar to a row in a SQL database table. Each point has a measurement, a tag set, a field key, a field value, and a timestamp.Point is uniquely identified by its series and timestamp.
- 在一个序列(series)中，不能存储多个具有相同时间戳的point。如果将point写入了一个包含相同时间戳point的序列，则字段集将成为旧字段集和新字段集的并集，并且任何连接都将转到新字段集

5. retention policy (RP)
- Describes how long InfluxDB keeps data (duration), how many copies of the data to store in the cluster (replication factor), and the time range covered by shard groups (shard group duration). RPs are unique per database and along with the measurement and tag set define a series.
- When you create a database, InfluxDB creates a retention policy called autogen with an infinite duration, a replication factor set to one, and a shard group duration set to seven days. For more information, see

6. series(序列)
- A logical grouping of data defined by shared measurement, tag set, and field key.
- 一个series对应一次插入操作（单行）。一个series会根据tag set和measurement被分为多条point。
7. timestamp
- The date and time associated with a point. All time in InfluxDB is UTC.

### glossary(术语) 

...

### schema design（概要设计）& data layout(数据布局)

1. 存储数据的位置（标记或字段）
- 将常见的查询和分组（group（）或group BY）元数据存储在标记中。
- 字段(fields)保存的值最好是唯一的
- 数字全部保存在字段(fields)中(tag只支持string类型)

2. 避免过多的序列(series)
- IndexDB 将measurement和tags作为索引

- 标记值被索引，字段值不被索引。这意味着，按标记查询比按字段查询性能更好。但是，当创建的索引过多时，写入和读取速度都可能开始减慢。

- 每个唯一的索引数据元素集形成一个序列键。包含高度可变信息（如唯一ID、哈希和随机字符串）的标记会产生大量序列，也称为高序列基数。高系列基数是许多数据库工作负载的高内存使用率的主要驱动因素。因此，为了减少内存消耗，请考虑将高基数值存储在字段值中，而不是存储在标记或字段键中。

## 2.x
### bucket
  1.x版本的influxdb有“数据库”的概念，在2.x中有类似的概念“bucket”。