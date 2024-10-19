

转载请注明出处：


#### oplog（操作日志）是MongoDB中用于记录所有写操作的日志。它是一个特殊的集合，存储在副本集的主节点中。oplog用于确保副本集中的副节点与主节点的数据保持一致。当主节点执行写操作时，相应的操作将被记录到oplog中，副节点则通过读取oplog来获取最新的数据变化。


#### 数据结构


　　oplog在mongo数据库主节点的local数据下，进入local数据库可以查看这个集合中存储数据的结构：


　　进入oplog数据库：




```
use local
```


　　查看数据结构：




```
rs0:PRIMARY> db.oplog.rs.findOne()
{
        "ts" : Timestamp(1729001735, 165),
        "t" : NumberLong(187),
        "h" : NumberLong(0),
        "v" : 2,
        "op" : "i",
        "ns" : "topology.filteredReason",
        "ui" : UUID("94a529f7-dbf1-4be0-b296-361094515e13"),
        "wall" : ISODate("2024-10-15T14:15:35.636Z"),
        "lsid" : {
                "id" : UUID("bbf4c6e6-e139-48e2-82df-a0b15bd0a89f"),
                "uid" : BinData(0,"47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=")
        },
        "txnNumber" : NumberLong(29170),
        "stmtId" : 0,
        "prevOpTime" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
        },
        "o" : {
                "_id" : "75a4d50f-8f37-4d06-8abe-d1c2b0de97a0",
                "reason" : "link[maipu_3:gigabitethernet3>maipu_2:gigabitethernet3] has no label",
                "type" : "LINK",
                "key" : "maipu_3:gigabitethernet3>maipu_2:gigabitethernet3",
                "tePolicyColorTupleId" : "maipu_3_maipu_1_MaiPu31_3628",
                "_class" : "com.tethrnet.terra.service.sr.topology.lib.model.terra.te.svc.filteredreasontop.FilteredReason"
        }
}
rs0:PRIMARY>
```


　　oplog是一个无限增长的集合，包含以下主要字段：


* ts：时间戳，表示操作的时间。
* h：哈希值，用于唯一标识该操作。
* op：操作类型，可能的值包括“i”（插入）、“u”（更新）、“d”（删除）。
* ns：命名空间，表示操作所属的数据库和集合。
* o：操作的详细内容，具体取决于操作类型。


#### 特性



1. oplog（操作日志）是一个特殊的固定大小集合，记录所有修改数据库数据的操作。

2. 它可以超越其配置大小限制，以避免删除大多数提交点。

3. 每个副本集成员都包含oplog的副本，位于local.oplog.rs集合中。

4. 操作日志中的每个操作都是幂等的，即应用一次或多次都会产生相同的结果。

5. oplog用于复制和恢复节点，如果节点下线，需要oplog来恢复状态。


#### 配置



1. 默认情况下，MongoDB会自动创建oplog，但可以自定义其大小。

2. 可以通过oplogSizeMB选项指定初始oplog大小。

3. 使用replSetResizeOplog命令动态更改oplog大小，无需重启mongod。

4. 可以设置最小保留时间，防止删除最近的操作日志条目。


#### oplog的作用


* 数据同步：副节点通过读取oplog来同步主节点的写入操作，确保数据一致性。
* 故障恢复：在主节点发生故障时，可以根据oplog中的信息快速恢复数据。
* 查询历史：oplog提供了一个操作历史的记录，有助于审计和问题排查。




#### 命令


　　**oplog存储在`local`数据库中。进入MongoDB shell后，您需要切换到`local`数据库：**



1. 查看当前最大oplog大小：


```
db.getSiblingDB("local").oplog.rs.stats(1024*1024).maxSize
```
2. 更改最大oplog大小：


```
db.adminCommand({
  replSetResizeOplog: 1,
  size: Double(16384) // megabytes
})
```
3. #### 过滤oplog日志


只对特定操作或特定时间范围内的日志，可以使用查询条件过滤结果。例如，查看类型为“插入”的操作：




```
db.oplog.rs.find({ op: "i" }).pretty()
```
4. 设置最小保留时间：


```
db.adminCommand({
  replSetResizeOplog: 1,
  minRetentionHours: 24 // 保留至少24小时的日志条目
})
```



#### 其他重要信息



1. oplog大小取决于存储引擎和可用空间。内存存储引擎使用5%物理内存，WiredTiger使用5%可用空间。

2. 默认情况下，不设置最小保留时间，系统会自动截断以维持配置的最大oplog大小。

3. 在副本集中运行时，不能手动对oplog执行写操作。

4. oplog对于复制和恢复节点至关重要，不能删除local.oplog.rs集合。

5. 在sharded集群中，每个副本组都有自己的oplog。


　　通过理解和正确配置oplog，您可以优化MongoDB的复制性能，并确保数据的一致性和可用性。适当的oplog大小对于高效的复制和恢复至关重要。



 


 


 



 本博客参考[豆荚加速器](https://yirou.org)。转载请注明出处！
