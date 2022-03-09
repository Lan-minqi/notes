1. ReadFromMergeTree::spreadMarkRangesAmongStreamsFinal()中为什么要加ExpressionTransform

  ```C++
  std::make_shared<ExpressionTransform>(header, sorting_expr);
  ```

  给后面的聚合算子提供排序列数据（而不是做了排序），查看StorageInMemoryMetadata结构代码和MergeTree使用方式，会发现MergeTree的排序方式定义为一个Expression
