# 数据结构

!!! abstract "笔记说明"
    这份笔记用于复习考研数据结构内容，参考王道数据结构、陈越数据结构

## ch1. 算法评价

!!! info "算法的五个重要特性"
    1. **输入**：算法在执行前需要外界提供0个或多个输入数据
    2. **输出**：算法在执行后至少产生一个输出数据
    3. **有穷性**：算法必须在执行有限步骤后终止
    4. **确定性**：算法的每一步都有确定的含义，不能有歧义
    5. **可行性**：算法的每一步都能通过有限的时间和空间来完成

### 时间复杂度

!!! tip "时间复杂度增长速度比较"
    $$ O(1) < O(\log n) < O(n) < O(n\log n) < O(n^2) < O(n^3) < O(2^n) < O(n!) < O(n^n) $$

!!! example "复杂度分析题目（2022年）"
    ```c
    int sum = 0;
    for (int i = 1; i < n; i *= 2)      // 外层循环执行 log₂n 次
        for (j = 0; j < i; j++)         // 内层循环执行 i 次
            sum++;
    ```
    
    **分析过程：**
    
    - 外层循环：i 依次为 1, 2, 4, 8, ..., n/2（约 log₂n 次）
    - 内层循环总执行次数：1 + 2 + 4 + 8 + ... + n/2 = n - 1
    - **时间复杂度：O(n)**

## ch2. 线性表

!!! abstract "线性表定义"
    具有相同数据类型的n个数据元素的有限序列，记作 L = (a₁, a₂, ..., aₙ)

### 2.1 基本操作

!!! note "线性表基本操作"
    1. `InitList(&L)` - 初始化线性表
    2. `Length(L)` - 求表长
    3. `LocateElem(L, e)` - 按值查找
    4. `GetElem(L, i)` - 按位查找
    5. `ListInsert(&L, i, e)` - 插入操作
    6. `ListDelete(&L, i, &e)` - 删除操作
    7. `PrintList(L)` - 输出操作
    8. `Empty(L)` - 判空操作
    9. `DestroyList(&L)` - 销毁操作

### 2.2 顺序表

!!! info "顺序表特点"
    一组地址连续的存储单元依次存储线性表中的数据元素，实现**逻辑相邻 = 物理相邻**

**优缺点对比：**

| 优点 | 缺点 |
|------|------|
| 存储密度高，随机存取 | 插入删除需要移动大量元素 |
| 访问速度快 | 空间利用率不高 |

!!! warning "顺序表操作复杂度"
    | 操作 | 最好情况 | 最坏情况 | 平均情况 |
    |------|----------|----------|----------|
    | 插入 | O(1) | O(n) | O(n) |
    | 删除 | O(1) | O(n) | O(n) |
    | 查找 | O(1) | O(n) | O(n) |
    | 按位访问 | O(1) | O(1) | O(1) |

### 2.3 链表

!!! info "链表结构定义"
    使用`data|next`结构存储线性表中的数据元素

```c
typedef struct LNode {
    ElemType data;          // 数据域
    struct LNode *next;     // 指针域
} LNode, *LinkList;
```

**链表特性：**

!!! note "链表特点"
    - **非随机存取**：必须从头结点开始遍历
    - **离散存储**：结点在内存中不连续
    - **动态分配**：可在运行时动态调整大小
    - **头结点**：便于统一处理（可选）

#### 链表基本操作

!!! example "链表初始化（带头结点）"
    ```c
    bool InitList(LinkList &L) {
        L = (LNode *)malloc(sizeof(LNode));  // 分配头结点
        if (L == NULL) return false;         // 内存不足，分配失败
        L->next = NULL;                      // 头结点指针域置空
        return true;
    }
    ```

!!! tip "插入操作技巧"
    - **前插法**：需要找到第i-1个结点p
    - **后插法**：可以直接通过第i个结点p执行插入（将新结点插入到p后面，然后交换数据）

#### 特殊链表

!!! info "双向链表"
    ```c
    typedef struct DNode {
        ElemType data;
        struct DNode *prior, *next;    // 前驱和后继指针
    } DNode, *DLinkList;
    ```

!!! info "循环链表"
    - **单向循环**：尾结点的next指向头结点
    - **双向循环**：头结点的prior指向尾结点，尾结点的next指向头结点

!!! info "静态链表"
    ```c
    typedef struct {
        ElemType data;
        int next;              // 下一个元素的数组下标
    } SLinkList[MAXSIZE];
    ```
    
    - 用数组模拟链表，适用于不支持指针的语言
    - `next == -1` 作为结束标志

## ch3. 栈、队列和数组

### 3.1 栈

!!! abstract "栈的定义"
    **LIFO**（Last In First Out，后进先出）的线性数据结构，只能在栈顶进行插入和删除操作

!!! note "栈的基本操作"
    1. `InitStack(&S)` - 初始化栈
    2. `StackEmpty(S)` - 判断栈是否为空
    3. `Push(&S, e)` - 入栈
    4. `Pop(&S, &e)` - 出栈
    5. `GetTop(S, &e)` - 读栈顶元素
    6. `DestroyStack(&S)` - 销毁栈

!!! tip "卡特兰数"
    n个不同元素进栈时，出栈元素的不同排列数为：
    $$C_n = \frac{1}{n+1} \binom{2n}{n}$$

#### 3.1.1 顺序栈

!!! example "顺序栈实现"
    ```c
    typedef struct {
        ElemType data[MAXSIZE];
        int top;                // 栈顶指针，初始值为-1
    } SqStack;
    
    // 判空条件：top == -1
    // 判满条件：top == MAXSIZE-1
    ```

!!! tip "共享栈"
    两个栈共享一个数组空间，提高内存利用率
    
    ```
    0                       MAXSIZE-1
    |       |       |       |
    bottom0 top0    top1   bottom1
    ```
    
    - `top0` 从左向右增长，`top1` 从右向左增长
    - 栈满条件：`top0 + 1 == top1`

#### 3.1.2 链式栈

!!! example "链式栈实现"
    ```c
    typedef struct LNode {
        ElemType data;
        struct LNode *next;
    } LNode, *LinkStack;
    ```
    
    - 栈顶指针指向链表头部
    - 无栈满问题，只要内存足够

### 3.2 队列

!!! abstract "队列定义"
    **FIFO**（First In First Out，先进先出）的线性数据结构，在队尾插入，在队头删除

```c
typedef struct {
    ElemType data[MAXSIZE];
    int front, rear;        // 队头和队尾指针
} SqQueue;
```

!!! note "队列基本操作"
    1. `InitQueue(&Q)` - 初始化队列
    2. `QueueEmpty(Q)` - 判断队列是否为空
    3. `EnQueue(&Q, e)` - 入队
    4. `DeQueue(&Q, &e)` - 出队
    5. `GetHead(Q, &e)` - 读队头元素
    6. `DestroyQueue(&Q)` - 销毁队列

#### 循环队列

!!! warning "循环队列的关键问题"
    如何区分队空和队满？三种解决方案：

!!! example "方案一：牺牲一个存储单元"
    ```c
    // 队空条件：front == rear
    // 队满条件：(rear + 1) % MAXSIZE == front
    // 元素个数：(rear - front + MAXSIZE) % MAXSIZE
    ```

!!! example "方案二：增加size变量"
    ```c
    typedef struct {
        ElemType data[MAXSIZE];
        int front, rear, size;
    } SqQueue;
    
    // 队空条件：size == 0
    // 队满条件：size == MAXSIZE
    ```

!!! example "方案三：增加tag标志"
    ```c
    typedef struct {
        ElemType data[MAXSIZE];
        int front, rear, tag;
    } SqQueue;
    
    // 队满：插入导致 rear == front 时置 tag = 1
    // 队空：删除导致 front == rear 时置 tag = 0
    ```

#### 链式队列

!!! example "链式队列实现"
    ```c
    typedef struct LinkNode {
        ElemType data;
        struct LinkNode *next;
    } LinkNode;
    
    typedef struct {
        LinkNode *front, *rear;    // 队头和队尾指针
    } LinkQueue;
    ```

!!! tip "链式队列特点"
    - 通常使用带头结点的单链表
    - 不存在队满问题
    - 入队在rear端，出队在front端

#### 双端队列

!!! info "双端队列（deque）"
    - **定义**：可以在队头和队尾进行插入和删除操作
    - **输入受限的双端队列**：只能在一端进行插入操作
    - **输出受限的双端队列**：只能在一端进行删除操作

### 3.3 栈和队列的应用

!!! note "应用场景"
    - **栈**：表达式求值、括号匹配、函数调用、递归实现
    - **队列**：层次遍历、缓冲区、打印队列、广度优先搜索

### 3.4 数组和特殊矩阵

!!! todo "待补充内容"
    数组存储和特殊矩阵的压缩存储

## ch4. 串

### 4.1 基本概念

!!! abstract "串的定义"
    **串（字符串）**：由零个或多个字符组成的有限序列，记作 S = "a₁a₂...aₙ"

!!! info "重要概念"
    - **空串**：长度为0的串
    - **子串**：串中任意多个连续字符组成的子序列
    - **主串**：包含子串的串
    - **字符在串中的位置**：字符在串中的序号

### 4.2 存储结构

!!! example "定长顺序存储"
    ```c
    #define MAXLEN 255
    typedef struct {
        char ch[MAXLEN];
        int length;
    } SString;
    ```

!!! example "堆分配存储"
    ```c
    typedef struct {
        char *ch;               // 动态分配存储区首地址
        int length;             // 串的长度
    } HString;
    ```

!!! note "块链存储"
    用固定大小的块链表存储，适合大串操作

!!! tip "参考实现"
    C语言具体实现可参考 `技术/cpp/cpp_basic.md`

### 4.3 KMP算法

!!! abstract "算法背景"
    解决字符串模式匹配问题，避免暴力匹配中的重复比较

#### 4.3.1 基本思想

!!! info "算法比较"
    - **暴力匹配**：主串和模式串逐字符比较，时间复杂度 O(mn)
    - **KMP算法**：利用模式串的"最长相等前后缀"信息，时间复杂度 O(m+n)

#### 4.3.2 next数组

!!! important "next数组的作用"
    在模式串失配时，指示下一个比较位置，避免主串指针回溯

!!! note "计算规则"
    - `next[j]` 表示模式串 `T[1..j]` 的最长相等前后缀长度
    - 失配时，j 跳转到 `next[j]` 位置继续比较

#### 4.3.3 代码实现

!!! example "next数组求解"
    ```c
    void get_next(SString T, int next[]) {
        int i = 1, j = 0;
        next[1] = 0;                    // 单个字符的最长前后缀长度为0
        
        while (i < T.length) {
            if (j == 0 || T.ch[i] == T.ch[j]) {
                ++i; ++j;
                next[i] = j;            // 匹配成功，长度加1
            } else {
                j = next[j];            // 失配，j回退
            }
        }
    }
    ```

!!! example "KMP主算法"
    ```c
    int Index_KMP(SString S, SString T, int pos, int next[]) {
        int i = pos, j = 1;             // i指向主串，j指向模式串
        
        while (i <= S.length && j <= T.length) {
            if (j == 0 || S.ch[i] == T.ch[j]) {
                ++i; ++j;               // 匹配成功，继续比较下一个
            } else {
                j = next[j];            // 失配，模式串指针回退
            }
        }
        
        if (j > T.length)               // 匹配成功
            return i - T.length;
        else
            return 0;                   // 匹配失败
    }
    ```

#### 4.3.4 算法优化

!!! tip "nextval数组优化"
    当 `T[j] == T[next[j]]` 时，可以直接跳转到 `nextval[j] = nextval[next[j]]`，进一步减少比较次数

!!! success "算法总结"
    - KMP通过next数组实现高效匹配，避免重复回溯
    - next数组的核心是利用模式串的前缀信息
    - 时间复杂度：O(m+n)，空间复杂度：O(m)

## ch5. 树与二叉树

!!! todo "待完善内容"
    树的基本概念、二叉树的性质与存储、遍历算法、线索二叉树、树与森林、哈夫曼树等

## ch6. 图

!!! todo "待完善内容"
    图的基本概念、存储结构、遍历算法、最小生成树、最短路径、拓扑排序、关键路径等

## ch7. 查找

!!! todo "待完善内容"
    线性查找、二分查找、分块查找、B树、B+树、散列表等

## ch8. 排序

!!! todo "待完善内容"
    插入排序、选择排序、交换排序、归并排序、基数排序、外部排序等

