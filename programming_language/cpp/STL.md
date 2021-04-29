# STL

## 同类容器对比

| 容器 | 底层实现 | key是否有序 | key是否可重复 | key是否可修改 | 查询效率 | 增删效率 | 模板参数 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| set | 红黑树 | 有序 | 不可重复 | 不可修改 | O(logn) | O(logn) | less \<T> |
| multiset | 红黑树 | 有序 | 可重复 | 不可修改 | O(logn) | O(logn) | less \<T> |
| unordered_set | 哈希表 | 无序 | 不可重复 | 不可修改 | O(1) | O(1) | hash \<T>, equal_to \<T> |
| map | 红黑树 | 有序 | 不可重复 | 不可修改 | O(logn) | O(logn) | less \<T> |
| map | 红黑树 | 有序 | 可重复 | 不可修改 | O(logn) | O(logn) | less \<T> |
| unordered_map | 哈希表 | 无序 | 不可重复 | 不可修改 | O(1) | O(1) | hash \<T>, equal_to \<T> |

## priority_queue

```cpp
    // 指定less function
    using Value = pair<int,int>;
    auto less = [](const Value &a, const Value &b)
    {
        return a.second < b.second;
    };
    priority_queue<Value, vector<Value>, decltype(less)> q(less);
```
