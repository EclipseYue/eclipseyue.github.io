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
    C语言具体实现可参考[cpp](../../Tech/cpp/cpp_basic.md)

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

### 5.1 树的基本概念

!!! abstract "树的定义"
    树是一个非线性数据结构，由结点和边组成，满足以下性质：
    
    - 有且仅有一个**根结点**
    - 当n>1时，其余结点可分为m(m>0)个互不相交的有限集T₁,T₂,...,Tₘ，称为根的**子树**

!!! important "树的特点"
    1. 树的根节点没有前驱，除根节点外的所有结点有且只有一个前驱
    2. 每个结点有0个或多个后继结点
    3. 在n个结点的树中，有且仅有n-1条边

#### 基本术语

!!! note "结点关系术语"
    - **祖先（ancestor）**：从根到该结点路径上的所有结点
    - **子孙（descendant）**：以该结点为根的子树中的所有结点
    - **兄弟（sibling）**：具有相同双亲的结点
    - **堂兄弟（cousin）**：双亲在同一层的结点

!!! note "结点分类术语"
    - **结点的度**：树的一个结点的孩子个数
    - **树的度**：树中结点的最大度数
    - **分支结点（branch node）**：又叫非终端结点，度大于0的结点
    - **叶结点（leaf node）**：度为0的结点，又称终端结点

!!! note "层次相关术语"
    | 术语 | 定义 | 说明 |
    |------|------|------|
    | **结点的层次** | 根结点为第1层，其子结点为第2层... | 从1开始计数 |
    | **结点的深度** | 从根结点到该结点的路径长度 | 根结点深度为0 |
    | **结点的高度** | 以该结点为根的子树的高度 | 叶结点高度为0 |
    | **树的高度** | 树中结点的最大层数 | 等于根结点的高度+1 |

!!! note "其他重要术语"
    - **有序树**：结点的子树从左到右有顺序，不能交换
    - **无序树**：结点的子树没有顺序要求
    - **路径**：从一个结点到另一个结点经过的结点序列
    - **路径长度**：路径上边的个数
    - **森林**：m(m≥0)棵互不相交的树的集合

#### 树的性质

!!! tip "树的重要性质"
    
    **性质1：** 结点数与边数关系
    
    树中结点数 = 所有结点的度数之和 + 1
    
    **性质2：** m叉树第i层结点数上界
    
    度为m的树中第i层至多有 $m^{i-1}$ 个结点（i≥1）
    
    **性质3：** m叉树结点总数上界
    
    高度为h的m叉树至多有 $\frac{m^h - 1}{m - 1}$ 个结点

!!! warning "高度计算公式"
    **性质4：** m叉树最小高度
    
    度为m、具有n个结点的树的最小高度为：
    $$h = \lceil \log_m(n(m-1)+1) \rceil$$
    
    **性质5：** m叉树最大高度
    
    度为m、具有n个结点的树的最大高度为：$h = n - m + 1$

!!! example "性质应用举例"
    对于一棵有100个结点的三叉树：
    
    - 最小高度：$h = \lceil \log_3(100 \times 2 + 1) \rceil = \lceil \log_3(201) \rceil = 5$
    - 最大高度：$h = 100 - 3 + 1 = 98$

### 5.2 二叉树
!!! abstract "二叉树定义"
    二叉树是一种特殊的树形结构，其特点是每个节点至多只有两棵保持左右顺序的子树。

!!! warning "二叉树和度为2的有序树的区别"
    - 度为2的树至少有3个结点，而二叉树可以为空
    - 度为2的有序树的孩子的左右次序是相对于另一个孩子而言的，而二叉树的左右子树是相对于父节点而言的

!!! info 几种特殊的二叉树
    - **满二叉树**: 高度为h，且有 $2^h - 1$ 个结点的二叉树，层序编号后，对于编号为i的结点，其左孩子编号为2i，右孩子编号为2i+1，其父亲编号为i/2（向下取整）。
    - **完全二叉树**: 高度为h，有n个结点的二叉树，当且仅当其每个结点都与满二叉树的结点一一对应时，称为完全二叉树。完全二叉树的特点是从左到右依次填充结点，最后一层可能不满。
      - 若i <= n/2(向下取整)，则结点i为分支结点，否则为叶结点。
      - 叶结点只可能在层次最大的两层上出现，对于最大层次的叶结点，都依次排列在该层最左边的位置上。
      - 若有度为1的结点，则最多只可能有一个，且该节点只有左孩子而无右孩子。
      - 按层序编号后，一旦出现某结点（编号为i）为叶结点或只有左孩子，则编号大于i的结点均为叶结点。
      - 若n为奇数，则每个分支结点都有左孩子和右孩子，若n为偶数，则编号最大的分支结点(编号为n/2)只有左孩子而无右孩子。
      - 正则二叉树：树中每个分支结点都有2个孩子，即树中只有度为0或2的结点。

!!! info "二叉树的性质"
    - **性质1**: 非空二叉树上的叶结点数等于度为2的结点数加1，即
      $$ n_0 = n_2 + 1 $$
      其中n₀为叶结点数，n₂为度为2的结点数。(通过入度证明)
    - **性质2**: 非空二叉树的第k层最多有2^(k-1)个结点(k≥1)。
    - **性质3**: 高度为h的二叉树至多有 $2^h - 1$ 个结点(h≥1)。
    - **性质4**: 对完全二叉树按层序编号，有以下关系:
      - 若i<=n/2(向下取整)，则结点i为分支结点，否则为叶结点，即最后一个分支结点的编号为n/2(向下取整)。
      - 叶结点只可能在层次最大的两层上出现。
      - 若有度为1的结点，则最多只可能有一个，且该节点只有左孩子而无右孩子。
      - 按层序编号后，一旦出现某结点（编号为i）为叶结点或只有左孩子，则编号大于i的结点均为叶结点。
      - 若n为奇数，则每个分支结点都有左孩子和右孩子，若n为偶数，则编号最大的分支结点(编号为n/2)只有左孩子而无右孩子。
      - 当i > 1时，结点i的双亲结点编号为i/2（向下取整）。
      - 若结点i有左右孩子，则左孩子编号为2i，右孩子编号为2i+1。
      - 结点i所在层次为k，则k = log₂i(向下取整) + 1。
    - **性质5**: 二叉树的高度h与结点数n的关系为
      $$ h = \lceil \log_2(n + 1) \rceil $$
      或者
      $$ h = \lceil \log_2(n) + 1 \rceil $$
      其中n为结点数。

**二叉树的存储结构**
!!! abstract "二叉树存储结构"
    二叉树的存储结构主要有两种：顺序存储和链式存储。

   - **顺序存储**: 采用数组存储二叉树，适用于完全二叉树。通过层序遍历将结点按顺序存入数组，空缺位置用NULL填充。优点是存取方便，缺点是浪费空间。
   - **链式存储**: 采用链表存储二叉树，每个结点包含数据域和左右孩子指针。适用于稀疏树，优点是节省空间，缺点是存取较慢。

````c
typedef struct BiTNode {
    ElemType data;              // 数据域
    struct BiTNode *lchild;     // 左孩子指针
    struct BiTNode *rchild;     // 右孩子指针
} BiTNode, *BiTree;
````

!!! warning
    在含有n个结点的二叉链表中，含有n+1个空链域。

### 5.3 二叉树的遍历
!!! abstract "二叉树遍历"
    二叉树的遍历是指按照一定的顺序访问二叉树的所有结点。

!!! note "遍历方式"
    - **前序遍历**: 根结点 -> 左子树 -> 右
    - **中序遍历**: 左子树 -> 根结点 -> 右子树
    - **后序遍历**: 左子树 -> 右子树 -> 根
    - **层序遍历**: 按层次从上到下、从左到右访问结点
> 前中后可以理解为根被遍历到的顺序

> 对于手写遍历，想象从根节点从上到下、从左到右环绕树，然后前序中序后序分别在左中右三个位置有标记，按照碰到标记的顺序作为顺序。

前三种对应的递归算法如下:
```c
void PreOrder(BiTree T) {
    if (T != NULL) {
        visit(T);  // 访问根结点
        PreOrder(T->lchild);      // 递归访问左子树
        PreOrder(T->rchild);      // 递归访问右子树
    }
}

void InOrder(BiTree T) {
    if (T != NULL) {
        InOrder(T->lchild);      // 递归访问左子树
        visit(T);  // 访问根结点
        InOrder(T->rchild);      // 递归访问右子树
    }
}

void PostOrder(BiTree T) {
    if (T != NULL) {
        PostOrder(T->lchild);    // 递归访问左子树
        PostOrder(T->rchild);    // 递归访问右子树
        visit(T);  // 访问根结点
    }
}
````

前三种对应的非递归算法如下:
```c
todo "非递归遍历"
```

**层序遍历**:
```c
void LevelOrder(BiTree T) {
    InitQueue(Q);  // 初始化队列
    BiTree p;
    EnQueue(Q, T);  // 将根结点入队
    while (!QueueEmpty(Q)) {
        DeQueue(Q, &p);  // 出队一个结点
        visit(p);  // 访问该结点
        if (p->lchild != NULL) EnQueue(Q, p->lchild);
        if (p->rchild != NULL) EnQueue(Q, p->rchild);
    }
}
```

**由遍历序列构造二叉树**
若已知中序序列，再给出其他三种遍历序列的其中一种，就可以唯一确定一棵二叉树。


* 前序 + 中序
每次从前序找到根结点，然后在中序中找到根结点的位置，将中序分为左子树和右子树，递归构造。


```c
BiTree CreateTree_PreIn(SString pre, SString in) {
    if (pre.length == 0 || in.length == 0) return NULL;
    BiTree T = (BiTree)malloc(sizeof(BiTNode));
    T->data = pre.ch[0];  // 前序的第一个字符是根结点
    int pos = LocateElem(in, T->data);  // 在中序中找到根结点位置
    // 构造左子树：
    // 前序序列中，根结点后面的 pos 个元素对应左子树的前序序列（即 pre[1..pos]），
    // 中序序列中，根结点左侧的 pos 个元素对应左子树的中序序列（即 in[0..pos-1]）
    T->lchild = CreateTree_PreIn(SubString(pre, 1, pos), SubString(in, 0, pos - 1));

    // 构造右子树：
    // 前序序列中，左子树之后的剩余元素对应右子树的前序序列（即 pre[pos+1..end]），
    // 中序序列中，根结点右侧的剩余元素对应右子树的中序序列（即 in[pos+1..end]）
    T->rchild = CreateTree_PreIn(SubString(pre, pos + 1), SubString(in, pos + 1));

    return T;
}
```


* 后序 + 中序
每次从后序找到根结点，然后在中序中找到根结点的位置，将中序分为左子树和右子树，递归构造。

```c
BiTree CreateTree_PostIn(SString post, SString in) {
    if (post.length == 0 || in.length == 0) return NULL;
    BiTree T = (BiTree)malloc(sizeof(BiTNode));
    T->data = post.ch[post.length - 1];  // 后序的
    int pos = LocateElem(in, T->data);  // 在中序中找到根结点位置
    // 构造左子树：
    // 后序序列中，根结点前面的 pos 个元素对应左子树的后序序列（即 post[0..pos-1]），
    // 中序序列中，根结点左侧的 pos 个元素对应左子树的中序序列（即 in[0..pos-1]）
    T->lchild = CreateTree_PostIn(SubString(post, 0, pos - 1), SubString(in, 0, pos - 1));
    // 构造右子树：
    // 后序序列中，左子树之后的剩余元素对应右子树的后序序列（即 post[pos..end-1]），
    // 中序列中，根结点右侧的剩余元素对应右子树的中序序列（即 in[pos+1..end]）
    T->rchild = CreateTree_PostIn(SubString(post, pos, post.length - 2), SubString(in, pos + 1));
    return T;
}
```


* 层序 + 中序
每次从层序找到根结点，然后在中序中找到根结点的位置，将中序分为左子树和右子树，递归构造。
```c
BiTree CreateTree_LevelIn(SString level, SString in) {
    if (level.length == 0 || in.length == 0) return NULL;
    BiTree T = (BiTree)malloc(sizeof(BiTNode));
    T->data = level.ch[0];  // 层序的第一个字符
    int pos = LocateElem(in, T->data);  // 在中序中找到根结点位置
    // 构造左子树：
    // 中序序列中，根结点左侧的 pos 个元素对应左子树的中序序列（即 in[0..pos-1]）
    SString leftIn = SubString(in, 0, pos - 1);
    // 从层序中筛选出左子树的结点
    SString leftLevel = FilterLevel(level, leftIn);
    T->lchild = CreateTree_LevelIn(leftLevel, leftIn);
    // 构造右子树：
    // 中序序列中，根结点右侧的剩余元素对应右子树的中序序列（即 in[pos+1..end]）
    SString rightIn = SubString(in, pos + 1);
    // 从层序中筛选出右子树的结点
    SString rightLevel = FilterLevel(level, rightIn);
    T->rchild = CreateTree_LevelIn(rightLevel, rightIn);
    return T;
}
```


#### 线索二叉树

!!! abstract "线索二叉树的概念"
    利用二叉树中的空指针域，存放指向该结点在某种遍历次序下的前驱和后继结点的指针。

**基本思想：** 在含有n个结点的二叉树中，有n+1个空指针域，可以利用这些空指针存放前驱或后继的线索。

**结点结构：**
```c
typedef struct ThreadNode {
    ElemType data;                     // 数据域
    struct ThreadNode *lchild, *rchild; // 左右孩子指针
    int ltag, rtag;                    // 左右线索标志
} ThreadNode, *ThreadTree;
```

**标志位说明：**
- `ltag = 0`：lchild指向左孩子；`ltag = 1`：lchild指向前驱
- `rtag = 0`：rchild指向右孩子；`rtag = 1`：rchild指向后继

##### 中序线索二叉树

**构造过程：**
```c
// 中序线索化递归函数
void InThread(ThreadTree &p, ThreadTree &pre) {
    if (p != NULL) {
        InThread(p->lchild, pre);        // 递归线索化左子树
        
        if (p->lchild == NULL) {         // 左子树为空，建立前驱线索
            p->ltag = 1;
            p->lchild = pre;
        }
        if (pre != NULL && pre->rchild == NULL) { // 建立后继线索
            pre->rtag = 1;
            pre->rchild = p;
        }
        pre = p;                         // 更新前驱结点
        
        InThread(p->rchild, pre);        // 递归线索化右子树
    }
}

// 主函数
void CreateInThread(ThreadTree &T) {
    ThreadTree pre = NULL;
    if (T != NULL) {
        InThread(T, pre);
        if (pre->rchild == NULL) {       // 处理最后一个结点
            pre->rtag = 1;
        }
    }
}
```

**遍历操作：**
```c
// 找到中序遍历的第一个结点
ThreadNode *Firstnode(ThreadNode *p) {
    while (p->ltag == 0)                 // 沿左孩子走到底
        p = p->lchild;
    return p;
}

// 找到中序遍历的后继结点
ThreadNode *Nextnode(ThreadNode *p) {
    if (p->rtag == 0)                    // 有右子树
        return Firstnode(p->rchild);     // 右子树的最左结点
    else
        return p->rchild;                // 直接后继
}

// 中序遍历线索二叉树
void InOrder_Thread(ThreadTree T) {
    for (ThreadNode *p = Firstnode(T); p != NULL; p = Nextnode(p))
        visit(p);
}
```

##### 其他线索二叉树

!!! note "先序和后序线索化特点"
    
    **先序线索二叉树：**
    - 后继查找：有左孩子则后继为左孩子，否则为右孩子，叶结点的后继由线索指出
    - 前驱查找：较复杂，通常在线索化过程中处理
    
    **后序线索二叉树：**
    - 后继查找规则：
        1. 若为根结点，后继为空
        2. 若为双亲的右孩子，或为双亲的左孩子且双亲无右子树，后继为双亲
        3. 若为双亲的左孩子且双亲有右子树，后继为双亲右子树中后序遍历的第一个结点

### 5.4 树和森林

#### 树的存储结构

**双亲表示法：**
```c
typedef struct {
    ElemType data;
    int parent;                          // 双亲位置
} PTNode;

typedef struct {
    PTNode nodes[MAXSIZE];
    int n;                              // 结点数
} PTree;
```

**孩子表示法：**
```c
typedef struct CNode {
    int child;                          // 孩子结点在数组中的位置
    struct CNode *next;
} CNode;

typedef struct {
    ElemType data;
    CNode *firstchild;                  // 第一个孩子
} CTBox;

typedef struct {
    CTBox nodes[MAXSIZE];
    int n, r;                          // 结点数和根的位置
} CTree;
```

**孩子兄弟表示法（二叉树表示法）：**
```c
typedef struct CSNode {
    ElemType data;
    struct CSNode *firstchild, *nextsibling; // 第一个孩子和右兄弟
} CSNode, *CSTree;
```

#### 树、森林与二叉树的转换

!!! tip "转换规则"
    
    **树转二叉树：**
    1. 加线：在所有兄弟结点之间加一条连线
    2. 去线：树中每个结点，只保留与第一个孩子结点的连线，删除与其他孩子的连线
    3. 层次调整：以树的根结点为轴心，将整棵树顺时针旋转一定角度
    
    **森林转二叉树：**
    1. 把每棵树转换为二叉树
    2. 第一棵二叉树不动，其余二叉树依次作为前一棵二叉树根结点的右子树
    
    **二叉树转树或森林：**
    1. 若二叉树非空，则根结点的右子树为森林，左子树为第一棵树
    2. 按照树转二叉树的逆过程进行

#### 树和森林的遍历

**树的遍历：**
- **先根遍历**：先访问根结点，再依次遍历各子树
- **后根遍历**：先依次遍历各子树，再访问根结点

**森林的遍历：**
- **先序遍历**：依次对森林中每棵树进行先根遍历
- **中序遍历**：依次对森林中每棵树进行后根遍历

!!! important "遍历等价性"
    - 树的先根遍历 ≡ 对应二叉树的先序遍历
    - 树的后根遍历 ≡ 对应二叉树的中序遍历
    - 森林的先序遍历 ≡ 对应二叉树的先序遍历
    - 森林的中序遍历 ≡ 对应二叉树的中序遍历

### 5.5 哈夫曼树和哈夫曼编码

!!! abstract "基本概念"
    - **路径长度**：从树中一个结点到另一个结点之间的分支数目
    - **结点的权**：给每个结点赋予的一个有意义的数值
    - **结点的带权路径长度**：从根结点到该结点之间的路径长度与该结点权的乘积
    - **树的带权路径长度（WPL）**：树中所有叶结点的带权路径长度之和

**哈夫曼树（最优二叉树）：** 在含有n个带权叶结点的二叉树中，带权路径长度最小的二叉树。

#### 哈夫曼树的构造

!!! example "哈夫曼算法"
    1. 初始化：由给定的n个权值构成n棵只有根结点的二叉树，得到森林F
    2. 选取和合并：在F中选取两棵根结点权值最小的树作为左右子树，构造一棵新的二叉树，新树根结点权值为左右子树权值之和
    3. 删除和加入：从F中删除被选取的两棵树，把新构造的树加入F中
    4. 重复步骤2-3，直到F中只剩一棵树为止

**算法实现：**
```c
typedef struct {
    int weight;                         // 权值
    int parent, lch, rch;               // 双亲、左孩子、右孩子下标
} HTNode, *HuffmanTree;

void CreateHuffmanTree(HuffmanTree &HT, int *w, int n) {
    if (n <= 1) return;
    int m = 2 * n - 1;                  // 哈夫曼树总结点数
    HT = new HTNode[m + 1];             // 0号单元未用
    
    // 初始化
    for (int i = 1; i <= m; ++i) {
        HT[i].parent = HT[i].lch = HT[i].rch = 0;
    }
    for (int i = 1; i <= n; ++i) {
        HT[i].weight = w[i-1];
    }
    
    // 构造哈夫曼树
    for (int i = n + 1; i <= m; ++i) {
        int s1, s2;
        Select(HT, i - 1, &s1, &s2);   // 选择权值最小的两个结点
        HT[s1].parent = HT[s2].parent = i;
        HT[i].lch = s1; HT[i].rch = s2;
        HT[i].weight = HT[s1].weight + HT[s2].weight;
    }
}
```

#### 哈夫曼编码

**前缀编码：** 任何一个字符的编码都不是另一个字符编码的前缀。

!!! tip "哈夫曼编码特点"
    - 哈夫曼编码是前缀编码
    - 哈夫曼编码是最优前缀编码，即平均编码长度最短
    - 编码过程：从叶结点到根结点逆向求编码
    - 译码过程：从根结点开始，逐位读入编码，到达叶结点输出字符

## ch6. 图
todo "待补充内容"

## ch7. 查找
todo "待补充内容"

## ch8. 排序
todo "待补充内容"
