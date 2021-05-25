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

google基础设施特点

1. Disaggregated storage(分解存储)
  - 数据不可变, 文件能打开并追加, 但不能修改已有内容. 一旦文件完成(finalized), 将不能做任何修改
  - 元数据操作如展示文件列表、打开文件会明显比本地文件系统高, 因为这些操作会触发到元数据服务器的RPC调用
  - 无本地存储, 所有持久化数据都在远端, 任何读写操作都要通过RPC实现
2. 共享计算资源
  - 垂直扩展有挑战. 相比跑几个大任务, 跑很多小任务能提高总体利用率
  - 经常有机器因为各种原因挂掉, 任务需要快速恢复, 小任务容易做到快速恢复
  - 机器配置 资源使用参差,使得任务性能不可预测. 运行在此调度设施上的分布式系统需要配备复杂的处理任务异常的策略, 例如随机任务失败、间歇性任务被驱逐而不可用
3. 

## 相关资料

<https://research.google/pubs/pub48388/>
<http://lambda-architecture.net/>
