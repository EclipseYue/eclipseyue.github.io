# STL容器速查表

## 顺序容器

### vector (动态数组)
**特性**: 动态大小, 随机访问, 尾部插入删除O(1), 中间插入删除O(n)

**常用操作**:
```cpp
vector<int> v;
v.push_back(x);           // 尾部插入
v.pop_back();             // 尾部删除
v[i] = x;                 // 随机访问
v.at(i);                  // 安全访问(带边界检查)
v.insert(it, x);          // 指定位置插入
v.erase(it);              // 删除指定位置
v.size();                 // 大小
v.empty();                // 是否为空
v.clear();                // 清空
v.resize(n);              // 重新设置大小
v.reserve(n);             // 预留容量
```

### list (双向链表)
**特性**: 双向链表, 任意位置插入删除O(1), 不支持随机访问

**常用操作**:
```cpp
list<int> l;
l.push_back(x);           // 尾部插入
l.push_front(x);          // 头部插入
l.pop_back();             // 尾部删除
l.pop_front();            // 头部删除
l.insert(it, x);          // 指定位置插入
l.erase(it);              // 删除指定位置
l.remove(x);              // 删除所有值为x的元素
l.sort();                 // 排序
l.reverse();              // 反转
l.unique();               // 去重(需先排序)
```

### deque (双端队列)
**特性**: 双端队列, 头尾插入删除O(1), 随机访问O(1)

**常用操作**:
```cpp
deque<int> dq;
dq.push_back(x);          // 尾部插入
dq.push_front(x);         // 头部插入
dq.pop_back();            // 尾部删除
dq.pop_front();           // 头部删除
dq[i] = x;                // 随机访问
dq.at(i);                 // 安全访问
```

## 关联容器

### set (有序集合)
**特性**: 红黑树实现, 自动排序, 元素唯一, 查找O(logn)

**常用操作**:
```cpp
set<int> s;
s.insert(x);              // 插入
s.erase(x);               // 删除值为x的元素
s.erase(it);              // 删除迭代器位置
s.find(x);                // 查找, 返回迭代器
s.count(x);               // 统计个数(0或1)
s.lower_bound(x);         // 第一个>=x的位置
s.upper_bound(x);         // 第一个>x的位置
s.begin(), s.end();       // 迭代器
```

### map (有序映射)
**特性**: 红黑树实现, key-value对, key唯一且自动排序, 查找O(logn)

**常用操作**:
```cpp
map<string, int> m;
m[key] = value;           // 插入/修改
m.insert({key, value});   // 插入
m.erase(key);             // 删除
m.find(key);              // 查找
m.count(key);             // 统计个数(0或1)
m.lower_bound(key);       // 第一个>=key的位置
m.upper_bound(key);       // 第一个>key的位置
```

### unordered_set (无序集合)
**特性**: 哈希表实现, 无序, 元素唯一, 平均查找O(1)

**常用操作**:
```cpp
unordered_set<int> us;
us.insert(x);             // 插入
us.erase(x);              // 删除
us.find(x);               // 查找
us.count(x);              // 统计个数(0或1)
us.bucket_count();        // 桶数量
us.load_factor();         // 负载因子
```

### unordered_map (无序映射)
**特性**: 哈希表实现, key-value对, key唯一, 无序, 平均查找O(1)

**常用操作**:
```cpp
unordered_map<string, int> um;
um[key] = value;          // 插入/修改
um.insert({key, value});  // 插入
um.erase(key);            // 删除
um.find(key);             // 查找
um.count(key);            // 统计个数(0或1)
```

## 容器适配器

### stack (栈)
**特性**: LIFO(后进先出), 基于deque实现

**常用操作**:
```cpp
stack<int> st;
st.push(x);               // 入栈
st.pop();                 // 出栈(无返回值)
st.top();                 // 栈顶元素
st.empty();               // 是否为空
st.size();                // 大小
```

### queue (队列)
**特性**: FIFO(先进先出), 基于deque实现

**常用操作**:
```cpp
queue<int> q;
q.push(x);                // 入队
q.pop();                  // 出队(无返回值)
q.front();                // 队首元素
q.back();                 // 队尾元素
q.empty();                // 是否为空
q.size();                 // 大小
```

### priority_queue (优先队列)
**特性**: 大顶堆(默认), 基于vector实现, 自动维护最大值在堆顶

**常用操作**:
```cpp
priority_queue<int> pq;                    // 大顶堆
priority_queue<int, vector<int>, greater<int>> pq2;  // 小顶堆

pq.push(x);               // 插入
pq.pop();                 // 删除堆顶(无返回值)
pq.top();                 // 堆顶元素
pq.empty();               // 是否为空
pq.size();                // 大小
```

## Lambda表达式

**语法**: `[capture](parameters) -> return_type { body }`

**捕获方式**:
- `[]`: 不捕获任何变量
- `[=]`: 按值捕获所有变量
- `[&]`: 按引用捕获所有变量
- `[var]`: 按值捕获var
- `[&var]`: 按引用捕获var
- `[=, &var]`: 按值捕获所有变量, 按引用捕获var

**常用示例**:
```cpp
// 基本语法
auto lambda = [](int x) { return x * 2; };

// 捕获外部变量
int factor = 3;
auto multiply = [factor](int x) { return x * factor; };

// STL算法中使用
vector<int> v = {3, 1, 4, 1, 5};
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });

// 自定义比较器
auto cmp = [](const pair<int,int>& a, const pair<int,int>& b) {
    return a.second < b.second;
};
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);
```

## 容器选择指南

| 需求 | 推荐容器 |
|------|----------|
| 随机访问 + 尾部操作 | vector |
| 频繁头尾操作 | deque |
| 频繁中间插入删除 | list |
| 需要排序的唯一元素 | set |
| key-value映射且需排序 | map |
| 快速查找唯一元素 | unordered_set |
| 快速key-value查找 | unordered_map |
| LIFO操作 | stack |
| FIFO操作 | queue |
| 需要优先级 | priority_queue |

## 迭代器与for循环遍历

### 迭代器类型

**迭代器分类**:
- **输入迭代器**: 只读, 单向
- **输出迭代器**: 只写, 单向  
- **前向迭代器**: 读写, 单向
- **双向迭代器**: 读写, 双向 (list, set, map)
- **随机访问迭代器**: 读写, 随机访问 (vector, deque)

**常用迭代器操作**:
```cpp
// 获取迭代器
auto it = container.begin();    // 指向第一个元素
auto it = container.end();      // 指向最后一个元素的下一位
auto it = container.rbegin();   // 反向迭代器, 指向最后一个元素
auto it = container.rend();     // 反向迭代器, 指向第一个元素的前一位

// 迭代器操作
++it;                          // 前进一位
--it;                          // 后退一位 (双向/随机访问迭代器)
it += n;                       // 前进n位 (随机访问迭代器)
it -= n;                       // 后退n位 (随机访问迭代器)
*it;                           // 解引用, 获取值
it->member;                    // 访问成员
```

### for循环遍历方式

#### 1. 传统for循环 (适用于vector, deque等支持随机访问的容器)

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 索引遍历
for (int i = 0; i < v.size(); ++i) {
    cout << v[i] << " ";
}

// 安全的索引遍历
for (size_t i = 0; i < v.size(); ++i) {
    cout << v[i] << " ";
}
```

#### 2. 迭代器for循环 (适用于所有容器)

```cpp
// vector示例
vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}

// set示例
set<int> s = {1, 2, 3, 4, 5};
for (auto it = s.begin(); it != s.end(); ++it) {
    cout << *it << " ";
}

// map示例
map<string, int> m = {{"a", 1}, {"b", 2}};
for (auto it = m.begin(); it != m.end(); ++it) {
    cout << it->first << ": " << it->second << endl;
}
```

#### 3. 范围for循环 (C++11) (适用于所有容器)

```cpp
// 基本用法
vector<int> v = {1, 2, 3, 4, 5};
for (int x : v) {              // 按值传递
    cout << x << " ";
}

for (const int& x : v) {       // 按const引用传递 (推荐读取)
    cout << x << " ";
}

for (int& x : v) {             // 按引用传递 (用于修改)
    x *= 2;
}

for (auto x : v) {             // 自动类型推导
    cout << x << " ";
}

for (const auto& x : v) {      // 自动类型推导 + const引用 (最常用)
    cout << x << " ";
}

// map的范围for循环
map<string, int> m = {{"a", 1}, {"b", 2}};
for (const auto& pair : m) {
    cout << pair.first << ": " << pair.second << endl;
}

// C++17结构化绑定
for (const auto& [key, value] : m) {
    cout << key << ": " << value << endl;
}
```

#### 4. 反向遍历

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 反向迭代器
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    cout << *it << " ";
}

// 索引反向遍历
for (int i = v.size() - 1; i >= 0; --i) {
    cout << v[i] << " ";
}

// 更安全的索引反向遍历
for (size_t i = v.size(); i > 0; --i) {
    cout << v[i-1] << " ";
}
```

### 不同容器的遍历示例

#### vector遍历
```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 方式1: 索引
for (size_t i = 0; i < v.size(); ++i) {
    cout << v[i] << " ";
}

// 方式2: 迭代器
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}

// 方式3: 范围for (推荐)
for (const auto& x : v) {
    cout << x << " ";
}
```

#### set遍历
```cpp
set<int> s = {5, 2, 8, 1, 9};

// 迭代器遍历 (自动排序)
for (auto it = s.begin(); it != s.end(); ++it) {
    cout << *it << " ";  // 输出: 1 2 5 8 9
}

// 范围for (推荐)
for (const auto& x : s) {
    cout << x << " ";
}
```

#### map遍历
```cpp
map<string, int> m = {{"apple", 5}, {"banana", 3}, {"orange", 7}};

// 迭代器遍历
for (auto it = m.begin(); it != m.end(); ++it) {
    cout << it->first << ": " << it->second << endl;
}

// 范围for
for (const auto& pair : m) {
    cout << pair.first << ": " << pair.second << endl;
}

// C++17结构化绑定 (推荐)
for (const auto& [key, value] : m) {
    cout << key << ": " << value << endl;
}
```

#### list遍历
```cpp
list<int> l = {1, 2, 3, 4, 5};

// 迭代器遍历
for (auto it = l.begin(); it != l.end(); ++it) {
    cout << *it << " ";
}

// 范围for (推荐)
for (const auto& x : l) {
    cout << x << " ";
}
```

### 遍历时的修改操作

#### 安全删除元素
```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 删除所有偶数 - 正确方式
for (auto it = v.begin(); it != v.end();) {
    if (*it % 2 == 0) {
        it = v.erase(it);  // erase返回下一个有效迭代器
    } else {
        ++it;
    }
}

// set/map的删除
set<int> s = {1, 2, 3, 4, 5};
for (auto it = s.begin(); it != s.end();) {
    if (*it % 2 == 0) {
        it = s.erase(it);
    } else {
        ++it;
    }
}
```

#### 修改元素值
```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 所有元素乘以2
for (auto& x : v) {          // 注意使用引用
    x *= 2;
}

// 使用迭代器修改
for (auto it = v.begin(); it != v.end(); ++it) {
    *it *= 2;
}
```

### 性能建议

1. **优先使用范围for循环**: 代码简洁, 性能好
2. **读取时使用const引用**: `for (const auto& x : container)`
3. **修改时使用引用**: `for (auto& x : container)`
4. **vector用索引也很高效**: 特别是需要索引值时
5. **避免在遍历时修改容器大小**: 可能导致迭代器失效

### 常见错误

```cpp
// 错误: 迭代器失效
vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 3) {
        v.erase(it);  // 错误! it失效了
        // ++it;      // 错误! 对失效迭代器操作
    }
}

// 错误: 拷贝开销大
vector<string> v = {"long string 1", "long string 2"};
for (auto x : v) {           // 错误! 发生拷贝
    cout << x << endl;
}
// 正确
for (const auto& x : v) {    // 正确! 使用引用
    cout << x << endl;
}
```

