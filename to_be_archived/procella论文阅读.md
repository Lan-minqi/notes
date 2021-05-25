# procella论文阅读

Procella: Unifying serving and analytical data at YouTube

## ABSTRACT

需求
> reporting and dashboarding, embedded statistics in pages, time-series monitoring, and ad-hoc analysis

- 通常的做法是为每种使用场景定制基础设施, 导致基础设施复杂、昂贵、更难维护
- SQL query engine Procella同时解决了这四个问题

## 1. INTRODUCTION

- reporting and dashboarding  
用户需要访问实时的详细的面板, 这要求查询引擎支持每秒数万次的固定查询, 保证低延迟(10ms量级), 查询可能使用过滤 聚合 集合操作和join(`filters, aggregations, set operations and
joins`)  
挑战在于数据量大(每天增加数万亿行), 要求访问实时数据

- embedded statistics  
嵌入式的实时统计, 比如视频的赞/点击, 会导致简单却基数很高的查询  
系统需要同时支持每秒数百万的实时更新及低延迟查询

- monitoring
有些查询和dashboarding类似  
查询量往往更低, 因为监控是给内部工程师使用的  
需要支持额外的数据管理功能: 老数据自动降采样和过期, 查询支持有效逼近函数 时间序列函数等

- ad-hoc analysis
复杂的临时查询用来做数据分析  
查询量少(10个/s), 延迟适度(几秒到几分钟不等), 查询复杂模式不可预测, 数据量大(数万亿)

YouTube原来使用不同的存储来分别满足这些需求, 高速增长下带来的问题:
1. 数据经过不同的ETL流程进入各个系统, 导致数据不一致加载慢、高开发和维护成本等问题
2. 迁移数据降低可用性, 提高学习成本.很多系统不完全支持SQL, 导致组织内部数据重复和访问问题
3. 一些组件在YouTube的数据规模下有性能、扩展性、效率等问题

Procella的功能:
1. 丰富的API  
几乎支持标准SQL所有用法  
一些有用的扩展, 近似聚合,处理复杂嵌套和重复schema, 用户自定义函数等
2. 高可扩展性  
分离计算与存储, 支持高效扩展(数千台服务器, 数百PB数据)
3. 高性能
使用尖端query execution技术以达到数百万QPS毫秒级延迟的效果
4. 数据新鲜度  
支持大量 低延迟的batch或streaming模式的数据摄取, lambda架构原生支持

## 2. ARCHITECTURE

### 2.1 google基础设施特点

1. Disaggregated storage(分解存储)
  - 数据不可变, 文件能打开并追加, 但不能修改已有内容. 一旦文件完成(finalized), 将不能做任何修改
  - 元数据操作如展示文件列表、打开文件会明显比本地文件系统高, 因为这些操作会触发到元数据服务器的RPC(Remote Procedure Call, 远程过程调用)
  - 无本地存储, 所有持久化数据都在远端, 任何读写操作都要通过RPC实现
2. 共享计算资源
  - 垂直扩展有挑战. 相比跑几个大任务, 跑很多小任务能提高总体利用率
  - 经常有机器因为各种原因挂掉, 任务需要快速恢复, 小任务容易做到快速恢复
  - 机器配置 资源使用参差,使得任务性能不可预测. 运行在此调度设施上的分布式系统需要配备复杂的处理任务异常的策略, 例如随机任务失败、间歇性任务被驱逐而不可用

### 2.2 procella的组成

#### 2.2.1 Data Storage

procella的数据逻辑上按表组织, 所有持久化数据保存在分布式文件系统Colossus中, 这使得计算和存储解耦, 处理数据的服务可以自由扩展, 无论底下的数据量是多少.

同一份数据可以被多个procella实例使用

#### 2.2.2 Metadata Storage

procella不使用B树二级索引, 而使用`zone maps, bitmaps, bloom filters, partition and sort keys`等轻量的二级结构

metadata server在query planning阶段提供这些结构信息

这些二级结构一部分由registration server在文件注册阶段收集, 一部分由data servers在query evaluation阶段lazy收集

#### 2.2.3 Table management

表管理通过标准DDL(Data Definition Language)命令实现, 这些命令被发送到RgS(registration
server)

用户能指定`column names, data types, partitioning
and sorting information, constraints, data ingestion method
(batch or realtime)`等选项以保证最佳的表布局

对于实时表, 可以指定如何`age-out, down-sample or compact`数据

#### 2.2.4 Batch ingestion

一个常见的做法是: 用户使用离线批处理生成数据, 再通过向RgS发DDL RPC来注册数据

在数据注册步骤, RgS抽取表到文件的映射, 以及file headers中的二级结构.
如果file headers中没有出现想要的索引信息, 就会让data servers来懒生成高昂代价的二级结构

RgS也负责检查data和元数据中schemas的兼容性, 它会精简、合并复杂的schemas

#### 2.2.5 Realtime ingestion

IgS(ingestion server)是实时数据进入系统的入口

IgS收到数据后, 可选得将其转化为与表结构对齐并追加写入WAL(write ahead log)中, 同时根据partition定义将数据发送到data servers, 会发送到多个data servers做冗余.

数据临时存放于内存的buffer中供查询使用.
Buffer会定期存档到文件系统以帮助恢复数据, 这里尽力而为的不会阻碍query访问数据

有后台任务对wal做compact

查询可以从内存Buffer中尚未持久化的部分和磁盘上的数据中读, 通过此种脏读模式查询 延迟可以到达秒甚至亚秒级, 还能保持数据一致性

#### 2.2.6 Compaction

compaction server会定期compact和repartition IgS写下的logs数据 成更大的分区柱状束

用户可自定义基于SQL的compact逻辑, 比如过滤 聚合 淘汰老数据 保留最近数据的策略来更细粒度地控制实时数据

在每个compact循环后, compaction server会通过RgS更新元数据

### 2.3 Query LifeCycle



## 相关资料

<https://research.google/pubs/pub48388/>
<http://lambda-architecture.net/>
