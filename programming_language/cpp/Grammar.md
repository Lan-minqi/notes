# 语法

## 构造函数

初始化列表的作用
1. 调用父类的构造函数
1. 初始化无默认构造函数的成员变量
```cpp
class StorageBox : public Box {
public:
    StorageBox(int width, int length, int height, Label label)
        : Box(width, length, height), m_label(label){}
private:
    Label m_label;
};
```
