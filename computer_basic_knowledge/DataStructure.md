# 数据结构

## 数组

数组是存放在连续内存空间上的相同类型数据的集合

- 下标从0开始
- 内存空间地址连续
- 插入/删除需要的时间复杂度为O(n)
- 二维数组中只有某一维的地址是连续的

定义二维数组

```cpp
vector<vector<int>> res(n, vector<int>(n, 0)); // 使用vector定义一个二维数组
```

vector和array容器的区别:

- 前者变长, 后者定长

## 链表

几种类型

1. 单链表
1. 双链表
1. 循环链表

复杂度分析

- 增/删为O(1)
- 查找为O(n)

适用于频繁增删的情形

## 哈希表

哈希表是根据关键码的值而直接进行访问的数据结构

一般哈希表都是用来快速判断一个元素是否出现集合里

### hash function

将key映射到内部数组的索引

### 哈希碰撞

两个key经过hash function的结果相同

解决办法

- 链表法

    内部数组里存放链表头结点, 碰撞的元素存放于同一条链表中

    要选择适当的哈希表的大小，这样既不会因为数组空值而浪费大量内存，也不会因为链表太长而在查找上浪费太多时间

- 线性探测法

    发生冲突后, 就不断从数组下一位找空位存或找匹配的

    要求tableSize > dataSize

## 栈

```txt
push(x) -- 元素 x 入栈
pop() -- 移除栈顶元素
top() -- 获取栈顶元素
empty() -- 返回栈是否为空
```

## 队列

```txt
push(x) -- 将一个元素放入队列的尾部。
pop() -- 从队列首部移除元素。
peek() -- 返回队列首部的元素。
empty() -- 返回队列是否为空。
```

### 单调队列

可以用双向队列deque实现

### 优先队列

## 二叉树

### 满二叉树

如果一棵二叉树只有度为0的结点和度为2的结点，并且度为0的结点在同一层上，则这棵二叉树为满二叉树

### 完全二叉树

在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置

堆是具有以下性质的完全二叉树：每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆；或者每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆

大顶堆：arr[i] >= arr[2i+1] && arr[i] >= arr[2i+2]  
小顶堆：arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]  

### 二叉搜索树

- 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 它的左、右子树也分别为二叉排序树

### 平衡二叉搜索树

又称AVL树

它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树

### 存储方式

- 链表
- 数组
父节点索引为i, 则左右孩子索引分别为`2*i+1, 2*i+2`

### 遍历方式

- 深度优先遍历

    可以用递归, 也可以迭代(用栈实现)
  - 前序遍历(中左右)
  - 中序遍历(左中右)
  
    ```cpp
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> results;
        stack<TreeNode*> s;
        // 栈保存遍历到的节点

        while (root != nullptr || !s.empty())
        {
            while (root != nullptr)
            {
                s.emplace(root);
                root = root->left;
            }

            root = s.top();
            s.pop();
            results.emplace_back(root->val);
            root = root->right;
        }

        return results;
    }
    ```

  - 后序遍历(左右中)
- 广度优先遍历

    用队列实现
  - 层次遍历

## 其他

### 双指针法

双指针法(快慢指针法)常用于数组和链表

### 时间复杂度

2.7GHz的机器每秒能执行2.7*10^9个时钟周期
对应到简单运算, 时间复杂度与1s内可处理n规模的关系:

```txt
- O(n) 5*10^8
- O(n^2) 2.25*10^4
- O(nlog(n)) 2*10^7
```

### int范围

```txt
INT_MAX 2147483647 (2*10^10量级)
INT_MIN -2147483648
```

## Reference

<https://mp.weixin.qq.com/s/X7R55wSENyY62le0Fiawsg>
<https://www.cnblogs.com/chengxiao/p/6129630.html>