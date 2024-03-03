现象是一个简单的语句
```
alter table T_HN_Main_SingleBidSection add  BS_CivilBuildingCategory varchar(10) null
GO
```
执行了上十个小时都没有还在执行中

后面删除了几个不相关的大表---数据量很多，列很大的表后，发现执行都是秒执行了
所以基本上可以确认就是这些数据导致系统操作执行很慢。
可能是这些表都是直接导入的索引关系什么都不明确，可能原因如下：

即使您平时不直接操作那几张数据量很大的表，它们仍然可能间接影响数据库的整体性能，原因如下：

1. **系统资源占用**：大表即使不被直接操作，也占用了大量的磁盘空间和可能占用了相当部分的内存（缓存）。这减少了其他操作可用的资源，尤其是当系统试图优化数据存储和访问时。

2. **自动维护开销**：SQL Server 会定期对数据库进行自动维护任务，如更新统计信息、检查数据库一致性或维护索引。这些操作对于大表来说会消耗更多资源，即便这些表没有被直接操作。这种维护开销可能降低了数据库在执行其他任务时的性能。

3. **锁资源和并发性**：数据库系统为了维护数据一致性和完整性，会对进行操作的数据施加锁。大表可能因为系统级的操作（如事务日志管理、锁管理）而间接影响到锁资源的使用和并发性能，尤其是在高并发环境下。

4. **查询优化器的选择**：SQL Server 的查询优化器在生成查询计划时，会考虑数据库中所有表的统计信息。大表的存在可能影响优化器的决策过程，即使查询不直接涉及这些表，优化器在评估执行计划时也可能需要考虑这些大表的统计信息，从而影响整体性能。

5. **I/O子系统压力**：大表可能导致数据库文件增大，增加了I/O子系统的压力。即使这些表不被频繁访问，数据库的备份和恢复操作也会因此变得更加耗时，间接影响到数据库的响应能力和可用性。

### 结论

尽管您可能不直接操作这些大表，它们仍然通过各种方式间接影响数据库的性能和资源利用率。理解这些影响有助于在设计和维护数据库时作出更加明智的决策，以确保系统的高效运行。在处理大数据量的表时，采用如数据分区、定期维护（比如索引重建和更新统计信息）等策略，可以帮助减轻它们对数据库性能的影响。

***

 大表的数据量巨大,即使不经常操作,也有可能会对数据库服务器性能产生影响,主要有以下原因:

1. 内存占用。SQL Server需要将表和索引数据缓存到内存中,数据量大的表会占用更多内存。即使不查询,也已在内存中占用资源。

2. 预读IO。SQL Server的预读机制会自动对数据和索引进行预读,防止查询时的IO等待。所以大表也会产生一定IO负载。

3. 后台任务。如自动更新统计信息等任务,需要扫描整个表来运行,大表的额外开销更大。

4. 查询编译与优化。SQL Server需要对查询进行编译与优化,判断查询是否可用已缓存计划等,大表查询的编译与优化代价更高。

5. 碎片和竞争。大表也更容易产生索引和数据碎片,导致更多IO,也可能与其他查询产生锁竞争。

所以大表本身就是数据库的“重负”,即使不 aktive 查询,也会在后台与系统资源进行争夺,可能导致整体数据库性能下降。需要对大表进行碎片整理、拆分等管理,减轻压力。

***
## **定位大表的方法

 ### 一：定位数据库中大表占用资源较高的常用方法有:

1. 查看表数据空间大小。使用 sp_spaceused 系统存储过程来查看各个表和索引的数据空间大小,排序定位大表。

2. 查看表行数。使用 sys.dm_db_partition_stats 视图来查看各表的行数,行数过大的即为大表。

3. 监控表 IO 消耗。开启 SQL Server 的 DMV 监控,通过 sys.dm_io_virtual_file_stats 等视图分析各表的 IO 读写情况。

4. 统计表索引碎片。大表碎片也更严重,使用 sys.dm_db_index_physical_stats 视图检测索引碎片情况。

5. 跟踪锁问题。如果存在锁争用可以通过扩展事件追踪锁情况,判断是否与大表相关。

6. 评估内存使用。从 buffer pool 中统计各表索引和数据占用内存,通常大表内存使用高。

综合使用这些方法可以直观地发现数据库中资源占用高的大表,作为优化点进行改善。

### 二 要定位 SQL Server 数据库中的大表，可以通过查询数据库的元数据来实现。下面是一些查询示例，这些查询可以帮助您找到数据库中最大的表，包括它们的行数、占用的空间等信息。

### 查询所有表及其大小

以下查询返回数据库中所有表的名称、行数、以及总空间（包括数据空间和索引空间）：

```sql
SELECT 
    t.NAME AS TableName,
    s.Name AS SchemaName,
    SUM(p.rows) AS RowCounts,
    SUM(a.total_pages) * 8 AS TotalSpaceKB, 
    SUM(a.used_pages) * 8 AS UsedSpaceKB, 
    (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS UnusedSpaceKB
FROM 
    sys.tables t
INNER JOIN      
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN 
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN 
    sys.allocation_units a ON p.partition_id = a.container_id
LEFT OUTER JOIN 
    sys.schemas s ON t.schema_id = s.schema_id
WHERE 
    t.NAME NOT LIKE 'dt%' 
    AND t.is_ms_shipped = 0
GROUP BY 
    t.Name, s.Name
ORDER BY 
    TotalSpaceKB DESC;

```

### 查询表的行数

如果您只对表的行数感兴趣，可以使用以下查询：

```sql
SELECT 
    t.NAME AS TableName,
    s.Name AS SchemaName,
    p.rows AS RowCounts
FROM 
    sys.tables t
INNER JOIN      
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN 
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
LEFT OUTER JOIN 
    sys.schemas s ON t.schema_id = s.schema_id
WHERE 
    t.NAME NOT LIKE 'dt%' 
    AND t.is_ms_shipped = 0
    AND i.OBJECT_ID > 255 
    AND i.index_id <= 1 -- 0:Heap, 1:Clustered
GROUP BY 
    t.Name, s.Name, p.Rows
ORDER BY 
    RowCounts DESC;
```

### 注意事项

- 上述查询可能需要一些时间来执行，尤其是在包含大量数据的数据库中。
- 这些查询提供了一个基本的起点，但您可能需要根据具体情况调整或扩展它们，以满足特定的需求。
- 在进行任何重大的数据库更改之前（如删除大表），务必确保已经评估了这些更改的潜在影响，并已经备份了相关数据。

通过分析查询结果，您可以确定哪些表是数据库中最大的，从而针对性地进行优化或清理，以提高数据库的整体性能。