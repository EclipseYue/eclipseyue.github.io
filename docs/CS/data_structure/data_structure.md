# 数据结构

!!! info "这份笔记用于复习考研数据结构内容，参考王道数据结构、陈越数据结构"

## ch1.算法评价

* 算法有以下五个重要特性:
    1. 输入:算法在执行前需要外界提供0个或多个输入数据。
    2. 输出:算法在执行后至少产生一个输出数据。
    3. 有穷性:算法必须在执行有限步骤后终止。
    4. 确定性:算法的每一步都有确定的含义，不能有歧义。
    5. 可行性:算法的每一步都能通过有限的时间和空间来完成。
* 时间复杂度比较:
    * $$ O(1) < O(logn) < O(n) < O(nlogn) < O(n^2) < O(n^3) < O(2^n) < O(n!) < O(n^n) $$

* 题目

!!! info "题目"
    ```C
    // 2022
    int sum = 0;
    for (int i = 1; i < n; i*=2)
        for (j = 0; j < i; j++)
            sum++;

    ```
    时间复杂度为 O(n)，因为外层循环执行 logn 次，内层循环执行 i 次，i 依次为 1,2,4,...,n/2，因此总的执行次数为 1 + 2 + 4 + ... + n/2 = n - 1 = O(n)。


## ch2.线性表

### 2.1 定义与操作
* 定义:具有相同数据类型的n个数据元素的优先序列

* 基本操作:
    1. InitList(&L)
    2. Length(L)
    3. LocateElem(L, e)
    4. GetElem(L, i)
    5. ListInsert(&L, i, e)
    6. ListDelete(&L, i, e)
    7. PrintList(L)
    8. Empty(L)
    9. DestroyList(&L)
### 2.2 顺序表
* 定义:一组地址连续的存储单元依次存储线性表中的数据元素，使得逻辑相邻等于物理相邻。
* 优缺点
    * 优点:存储密度高，随机存取，访问速度快。
    * 缺点:插入和删除操作需要移动大量元素，空间利用率不高。
* 基本操作:
    1. InitList(&L)
        * 初始化顺序表长度，根据动态与否分配一段连续的存储空间。
    2. ListInsert(&L, i, e)
        * 插入元素e到顺序表L的第i个位置，i从1开始计数。最好最坏平均分别为 O(1), O(n), O(n)。
    3. ListDelete(&L, i, e)
        * 删除顺序表L的第i个位置的元素，并将其值赋给e。最好最坏平均分别为 O(1), O(n), O(n)。
    4. LocateElem(L, e)
        * 查找顺序表L中值为e的元素，返回其位置。最好最坏平均分别为 O(1), O(n), O(n)。
    5. GetElem(L, i)
        * 获取顺序表L的第i个位置的元素。自然为 O(1)。
### 2.3 链表
* 定义:使用data|next结构存储线性表中的数据元素，data存储数据元素，next存储下一个元素的地址。
    ```
    typedef struct LNode {
        ElemType data;
        struct LNode *next;
    } LNode, *LinkList;
    ```
* 特性:非随机存取，离散，遍历，可带头结点（头结点不存储数据，可存表长）。
* 解决问题:插入删除O(1)，头结点使得第一个点不特殊(不必头指针处理)，便于处理。
* 基本操作:
    1. InitList(&L)
        ```
        L = (LNode *)malloc(sizeof(LNode));
        L->next = NULL;
        return true;
        ```
    2. Length(L)
    3. GetElem(L, i)
    4. LocateElem(L, e)
    5. ListInsert(&L, i, e)
        * 前插法需要找到第i-1个结点p，而后插法可以直接通过第i个结点p执行插入
    6. ListDelete(&L, i, e)
    7. List_HeadInsert(&L, e)
    8. List_TailInsert(&L, e)
* 双向链表
    ```
    typedef struct DNode {
        ElemType data;
        struct DNode *prior, *next;
    } DNode, *DLinkList;
    ```
    * 操作:插入删除
* 循环链表
    * 单向循环链表:尾结点的next指向头结点
    * 双向循环链表:头结点的prior指向尾结点，尾结点的next指向头结点
* 静态链表
    * 用数组模拟链表，数组下标作为指针使用
    * 适用于结点频繁插入删除的场景
    * 需要一个备用链表来存储空闲结点
    ```
    typedef struct {
        ElemType data;
        int next; // 下一个元素的数组下标
    } SLinkList[MAXSIZE];
    ```
    * next == -1 作为结束标志

## ch3.栈、队列和数组

### 3.1 栈
* 定义: LIFO(后进先出)数据结构
* 特性: 只能在栈顶进行插入和删除操作
* 基本操作:
    1. InitStack(&S)
    2. StackEmpty(S)
    3. Push(&S, e)
    4. Pop(&S, &e)
    5. GetTop(S, &e)
    6. DestroyStack(&S)
* 特性: $$n$$ 个不同元素进栈时，出栈元素的不同排列数为 $$C_n = \dfrac{1}{n+1} \binom{2n}{n}$$（卡特兰数）

#### 3.1.1 栈的顺序存储
* 定义:使用一段地址连续的存储单元依次存储栈中的数据元素，top指向栈顶元素,初始值为-1.
    ````
    typedef struct {
        ElemType data[MAXSIZE];
        int top; // 栈顶指针
    } SqStack;
    ````
* 操作:
    1. InitStack(&S)
    2. StackEmpty(S)
    3. Push(&S, e)
    4. Pop(&S, &e)
    5. GetTop(S, &e)

* 共享栈
    ````
    0                       MAXSIZE-1
    |       |       |       |
bottom 0   top0    top1   bottom 1
    ````
    * top0从左向右增长，top1从右向左增长
    * top0 + 1 == top1 时栈满
    * top0 == -1 时栈0空，top1 == MAXSIZE 时栈1空

* 链式栈
    * 定义:使用链表存储栈中的数据元素，LHead指向栈顶元素。
    ````
    typedef struct LNode {
        ElemType data;
        struct LNode *next;
    } LNode, *LinkStack;
    ````
### 3.2 队列
* 定义: FIFO(先进先出)数据结构
````
typedef struct {
    ElemType data[MAXSIZE];
    int front, rear; // 队头和队尾指针
} SqQueue;
````
* 特性: 只能在队头(Front)进行删除操作，在队尾(Rear)进行插入操作
* 基本操作:
    1. InitQueue(&Q)
    2. QueueEmpty(Q)
    3. EnQueue(&Q, e)
    4. DeQueue(&Q, &e)
    5. GetHead(Q, &e)
    6. DestroyQueue(&Q)
* 问题:
    * 队列满: rear == MAXSIZE 产生"假队列溢出"
    * 队列空: front == rear
* 循环队列
    * 初始时: front == rear == 0
    * 队首指针进1: front = (front + 1) % MAXSIZE
    * 队尾指针进1: rear = (rear + 1) % MAXSIZE
    * 队列长度: (rear - front + MAXSIZE) % MAXSIZE
    * 出队入队:指针顺时针加一
    * 队列空: front == rear
    * 三种处理方式:
      * 牺牲一个单元区分队空和满，入队时少用一个队列单元
        * 队满: (rear + 1) % MAXSIZE == front
        * 队空: front == rear
        * 元素个数: (rear - front + MAXSIZE) % MAXSIZE
      * 增加size变量记录队列长度
        * 队满: size == MAXSIZE
        * 队空: size == 0
        * 元素个数: size
      * 新加tag区分队空和满
        * 队满: 插入导致rear == front 置tag == 1
        * 队空: 删除导致front == rear 置tag == 0
        * 元素个数: (rear - front + MAXSIZE) % MAXSIZE

        ![](https://eclipseyue-1323281044.cos.ap-nanjing.myqcloud.com/pic/REqueue.jpg)
        
* 队列的链式存储
    * 定义:使用链表存储队列中的数据元素，QHead指向队头元素，QTail指向队尾元素。
    ````
    typedef struct LinkNode {
        ElemType data;
        struct LinkNode *next;
    } LinkNode;
    typedef struct {
        LinkNode *front, *rear; // 队头和队尾指针
    } LinkQueue;
    ````
    * 不带头结点时,front == rear == NULL,则队列空。但是常用头结点单链表


* 双端队列
    * 定义:可以在队头和队尾进行插入和删除操作
    * 输入/输出受限的队列:只能在一端进行插入操作，或只能另一端进行删除操作
    * !!!这里需要做一下题


### 3.3 应用
记录几道题：
!!!做题
### 3.4 数组和特殊矩阵
!!!做题

## ch4.串


## ch5.树与二叉树

## ch6.图

## ch7.查找

## ch8.排序

