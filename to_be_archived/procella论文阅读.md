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
