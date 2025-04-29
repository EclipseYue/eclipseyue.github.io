# 编译原理
## 课程要求
* Homework=10% 一定要交！
* ClassQuizzes=10% 记得来！
* MidExam=15%. 努力复习拿高分。
* Labs=25% 布置后马上开始，多花时间，积极向助教寻求帮助，不要放弃。
* FinalExam=40% 平时分拿到40分以上，期末压力会小非常多。
* **FinalExam < 40/100 ➔Final Grade < 60/100**




## Chapter 2:

DFA NFA LL(0) LR(0) LL(1) LR(1) SLR LALR



## Chapter 5

### Symbol Table

* Symbol Table要干什么？

记录变量的类型、作用域。

1. 加入定义
2. 右定义覆盖左定义
3. 到作用域末尾，区域binding丢掉

* 支持四个操作:
insert
lookup
beginScope
endScope

* Multiple Symbol Table

像Java那样，支持多重作用域

而对于ML语言，则是没有前项引用，就累加下去

* 两种方式
Imperative Style（命令式）

从外面插入，从里面删除

命令式风格的方法则不需要申请多个符号表，自始至终在单个符号表上进行动态维护。其中每一项对应的不是一个变量的定义，而是变量定义的栈。

Functional Style（函数式）

函数式风格的方法在每次进入新的作用域时都需要申请一个新的符号表，而当离开该作用域时，就可以将该符号表销毁，相当于维护了一个符号表的栈。



* 怎么实现高效

1. 命令式

使用哈希表进行新的binding插入，然后新插入的在左边

insert:做哈希得到index，相应位置插入binding

lookup:做哈希得到index，从index开始向下查找，直到找到binding

pop:做哈希得到index，hash表位置变为下一个

2. 函数式

核心在于使得$\sigma$不被改变


![pic1](https://eclipseyue-1323281044.cos.ap-nanjing.myqcloud.com/pic/pic.jpg)

也就是复制数组，使得新的作用域有新的binding。但是这样太大、太浪费，所以使用二叉搜索树。

Insert:Copy the nodes from the root to the parent of the inserted node

LookUp:从根节点开始，向下查找 O(logn)

> 总结来说就是命令式插入毁原来表，进入新scope需要额外标记来删，
> 函数式插入新环境拿一份旧环境走，退出回到旧环境

### Symbol Table in Tiger

interface & implementation

```java
typedef struct S_symbol_ *S_symbol;
S_symbol S_symbol (string); /* Symbol */ string S_name(S_symbol);
typedef struct TAB_table_ *S_table;
S_table S_empty( void);
void S_enter( S_table t,S_symbol sym, void *value); void *S_look( S_table t, S_symbol sym);
void S_beginScope( S_table t);
void S_endScope( S_table t);

static S_symbol mksymbol (string name , S_symbol next) { 
    S_symbol s = checked_malloc(sizeof(*s));
    s->name = name; s->next = next;
    return s;
}
S_symbol S_symbol (string name) {
   int index = hash(name)%SIZE;
   S_symbol syms = hashtable[index], sym;
   for ( sym = syms; sym; sym = sym->next)
     if (0 == strcmp(sym->name, name)) return sym;
   sym = mksymbol(name,syms);
   hashtable[index] = sym;
   return sym;
}
string S_name (S_symbol sym) {
  return sym->name;
}

// make a new S_Table
S_table S_empty(void) {
  return TAB_empty();
}
// insert a binding
void S_enter(S_table t, S_symbol sym, void *value){ TAB_enter(t,sym,value);
}
// look up a symbol
void *S_look(S_table t, S_symbol sym) {
  return TAB_look(t,sym);
}
static struct S_symbol_ marksym = { “<mark>”, 0 };
void S_beginScope ( S_table t) {
  S_enter(t, &marksym, NULL);
}
void S_endScope( S_table t) {
  S_symbol s;
  do
    s= TAB_pop(t);
  while (s != &marksym);
}

```

使用辅助栈便于恢复那个最近的symbol
```java
struct TAB_table_ {
  binder table[TABSIZE];
  void *top;
};
t->table[index] = Binder(key, value, t->table[index], t->top);
static binder Binder(void *key, void *value, binder next, void *prevtop) {
  binder b = checked_malloc(sizeof(*b));
  b->key = key; b->value=value; b->next=next;
  b->prevtop = prevtop;
  return b;
}
```

## Type Checking

三个关键问题

1. Valid的类型表达式
2. 两个类型是否相同
3. 类型检查规则

### Types
* 基本类型
int, string
* 构建类型
由其他类型组成的records和arrays



Name Eq & Structural Eq

Tiger uses name Eq

只认名字不认结构

Environments for Type Checking

Type environment:
    symbol -> Ty_ty

Value environment:

    symbol -> Ty_ty
    symbol -> struct{Ty_tyList formals, Ty_ty result;}


Semant module实现了语义分析&类型检查

函数如下

```Java
Struct expty transVar (S_table venv, S_table tenv, A_var v); Struct expty transExp (S_table venv, S_table tenv, A_exp a); Void transDec (S_table venv, S_table tenv, A_dec d);
Ty_ty transTY (S_table tenv, A_ty a);
```

**transExp** recursive function 

实现了type-checking和IR Generation

### Type-Checking for Tiger

TransExp:类型检查表达式，查询和更新环境

可以在给定的两个环境下将输入的表达式标记上type（如果发现非法则报错）

接下来讲一个简单的"+"的类型检查


一些类型检查

#### Variable declarations


#### Type declarations


#### Function declarations


#### Recursive declarations

对于例子 ```type list = {first: int, rest: list}```

use 2 passes
1. 先把所有的类型(headers)都放到一个表里
2. 然后再进行body的类型检查

对于函数也是一样的



在此提个要求：手搓整个流程和各个中间件搞清楚（写完删）





## Chapter 6

Activation Records

当一个活动被调用的时候我们应该构建一个什么样的数据结构来保存它的状态

LIFO使用control stack来存储

每次当过程被调用，它的local variables和参数都被压入栈中
当过程返回的时候，栈顶的所有数据都被pop掉
这也被叫做活动过程 每一个live activity都对应一个activation record也叫做frame，在control stack中存储

实现为简单的栈是不够的，由于其批量性

垃圾 | 已分配



一个问题 Para Passing中出现覆盖

1. 好
什么情况


引用传递 修改这些东西


Static Link


Display

Lambda Lefting


Higher Order Functions

### 6.2 Frames in Tiger Compiler

escape与否?

* 逃逸分析

出不出作用域

一些接口

不考察不需要tiger 主要是给自己的实现思路

## Chapter 7


![location](https://eclipseyue-1323281044.cos.ap-nanjing.myqcloud.com/pic/location.png)
以下是完善后的讲稿内容，针对三地址码（Three-Address Code, TAC）及其相关概念进行了更清晰的组织和补充说明：

---

### 三地址码（Three-Address Code, TAC）

#### 基本形式
三地址码的基本形式如下：
```
x = y op z
```
其中：
- `x` 是目标变量（通常是临时变量）。
- `y` 和 `z` 是操作数（可以是常量、变量或临时变量）。
- `op` 是操作符（如加法、减法、乘法等）。

三地址码的特点是：右侧最多只有一个操作符。对于复杂的表达式，需要将其分解为多个简单的三地址指令。

#### 示例
例如，表达式 `2*a + (b-3)` 可以分解为以下三地址码：
```plaintext
t1 = 2 * a
t2 = b - 3
t3 = t1 + t2
```

#### 四字段结构
每条三地址码指令通常由四个字段组成：
1. **操作符（op）**：表示具体的操作（如加法、减法等）。
2. **三个地址字段**：分别用于存储操作数和结果的目标地址。

#### 其他形式
除了基本的二元操作外，三地址码还可以表示其他形式的操作，例如：
- **单目操作**：`t2 = -t1`
- **函数调用**：`CALL(f, args)`
- **内存访问**：`MEM(e)` 表示从内存中取值。

总之，三地址码没有严格的标准形式，但其核心思想是通过简单指令来表示复杂的计算过程。

---

### 中间表示（Intermediate Representation, IR）

三地址码是一种典型的中间表示（IR），用于编译器的中间阶段。以下是常见的 IR 表达式类型及其含义：

#### 表达式（Exp）
1. **CONST(i)**  
   表示整数常量 `i`。例如：`CONST(5)` 表示常量 `5`。

2. **NAME(s)**  
   表示符号名称 `s`。例如：`NAME(x)` 表示变量 `x`。

3. **TEMP(t)**  
   表示临时变量 `t`。例如：`TEMP(t1)` 表示临时变量 `t1`。

4. **BINOP(op, e1, e2)**  
   表示二元操作符 `op` 对两个表达式 `e1` 和 `e2` 的操作。例如：`BINOP(+, TEMP(t1), CONST(5))` 表示 `t1 + 5`。

5. **MEM(e)**  
   表示从内存地址 `e` 中取值。例如：`MEM(TEMP(t1))` 表示从 `t1` 指向的内存地址读取值。

6. **CALL(f, args/l)**  
   表示函数调用，其中 `f` 是函数名，`args` 是参数列表。例如：`CALL("printf", [TEMP(t1)])` 表示调用 `printf(t1)`。

7. **ESEQ(stm, exp)**  
   表示语句和表达式的组合。先执行语句 `stm`，然后计算表达式 `exp`。

有返回值的T_exp,没有返回值的T_stm,带有布尔值的表达式a conditional jump

![eg1](https://eclipseyue-1323281044.cos.ap-nanjing.myqcloud.com/pic/expeg1.png)

上图就是一个复杂语句拆分的例子



#### 语句（Stm）
1. **MOVE(TEMP t/dst, e/src)**  
   表示赋值操作，将表达式 `e` 的值赋给变量 `t`。例如：`MOVE(TEMP(t1), BINOP(+, TEMP(t2), CONST(3)))` 表示 `t1 = t2 + 3`。

2. **MOVE(MEM(e_1), e_2)**  
   表示将表达式 `e_2` 的值存储到内存地址 `e_1` 中。例如：`MOVE(MEM(TEMP(t1)), CONST(5))` 表示将 `5` 存储到 `t1` 指向的内存地址。

3. **EXP(e)**  
   表示将表达式 `e` 转换为语句。例如：`EXP(CALL("printf", [TEMP(t1)]))` 表示调用 `printf(t1)` 并忽略返回值。

4. **JUMP(e, labs)**  
   表示无条件跳转到标签 `labs`。例如：`JUMP(NAME("label1"))` 表示跳转到标签 `label1`。

5. **CJUMP(o, e1, e2, t, f)**  
   表示条件跳转。如果 `e1 o e2` 为真，则跳转到标签 `t`；否则跳转到标签 `f`。例如：`CJUMP(>, TEMP(t1), CONST(0), NAME("label1"), NAME("label2"))` 表示如果 `t1 > 0`，则跳转到 `label1`，否则跳转到 `label2`。

6. **SEQ(s1, s2)**  
   表示顺序执行语句 `s1` 和 `s2`。例如：`SEQ(MOVE(TEMP(t1), CONST(5)), MOVE(TEMP(t2), CONST(10)))` 表示依次执行 `t1 = 5` 和 `t2 = 10`。

7. **LABEL(n)**  
   表示标签 `n`，用于标记代码中的跳转目标。例如：`LABEL("label1")` 表示标签 `label1`。


---

### 条件跳转示例

条件跳转是控制流的一种重要形式。例如，表达式 `a > b || c < d` 可以翻译为以下三地址码：

```plaintext
t1 = a > b
if t1 goto label_true
t2 = c < d
if t2 goto label_true
goto label_false
label_true:
    // 执行 true 分支
label_false:
    // 执行 false 分支
```

#### 翻译过程
1. 计算 `a > b` 的结果，存入临时变量 `t1`。
2. 如果 `t1` 为真，跳转到 `label_true`。
3. 否则，计算 `c < d` 的结果，存入临时变量 `t2`。
4. 如果 `t2` 为真，跳转到 `label_true`。
5. 如果两个条件都为假，跳转到 `label_false`。

---

### 总结

三地址码是编译器中一种重要的中间表示形式，具有以下特点：
- 简单易懂，便于分析和优化。
- 支持复杂表达式的分解。
- 提供了丰富的语句和表达式类型，能够表示各种高级语言结构。

通过掌握三地址码及其相关概念，可以更好地理解编译器的工作原理，并为后续的优化和代码生成打下基础。






## Chapter 8

IR规范化 转化成规范树 Basic Blocks and Traces

Canonical Trees

1. No SEQ or ESEQ
2. The parent of each CALL is eithrt a MOVE or an EXP

一些问题：
1.CJUMP实际上在汇编中如果没符合条件会继续执行
2.ESEQ中的stmt会带来副作用
3.CALL也会引起同样的问题

E.g., e1: a + 5, s: a = 5
假设有一个表达式 e1: a + 5 和一个语句 s: a = 5。
如果我们将 s 包含在 e1 中，形成一个新的表达式 a + (a = 5)，那么这个表达式的值将取决于 a = 5 的执行时机。
如果先执行 a = 5，再计算 a + 5，结果将是 10；但如果先计算 a + 5，再执行 a = 5，结果将是原来的 a 加上 5。

4. 不能有两个CALL，因为都使用相同的寄存器来传递参数，就可能出现参数值被覆盖的情况。

规范树：
重写成规范树：
1. 不能有SEQ和ESEQ
2. 每个CALL的父节点是MOVE或者EXP

特性：
1. 每个标准树最多有一个stmt比如根节点，其他都是表达式
2. CALL的子节点不能是CALL
推论：CALL的父节点必须是标准树根节点，最多可以有一个CALL因为EXP和MOVE仅仅可以有一个CALL

要干三件事：
eliminate ESEQs
move CALLs to TOP level
eliminate SEQs

然后是过程




Taming Conditional Branches






## Chapter 9






## Chapter 10

Liveness Analysis

为了减少寄存器使用

未来会用到，则其为活跃

注意来(静态分析都活跃)和更新赋值后边

一些定义:out-edges,in-edges,pred[n],succ[n],def[n],use[n]
* out-edges:从n出发的边
* in-edges:到达n的边
* pred[n]:n的前驱
* succ[n]:n的后继
* def[n/a]:n定义的变量或者a被定义的语句集合
* use[n/a]:n使用的变量或者a被使用的语句集合

Liveness: a var is live on an edge if there is a directed path from that edge to a use of that var thatdoes not go through any def

Live-in: a var is live-in at a node if it is live on ant of the in-edges of that node
Live-out: a var is live-out at a node if it is live on any of the out-edges of that node

in[n]: Live-in set at node n
out[n]: Live-out set at node n 也就是其在某一个节点的后继节点的in set中

cal of liveness:

* $$Rule1:If a \in in[m] for any m \in succ[n],then a \in out[n]$$
* $$Rule2:If a \in use[n], then a \in in[n]$$
* $$Rule3:If a \in out[n] and a doesn't belong to def[n],then a \in in[n]$$

e.g.
![eg1](https://eclipseyue-1323281044.cos.ap-nanjing.myqcloud.com/pic/livenesseg1.png)

in[m1] = {d} in[m2] = {e}
out[n] = {d,e}
in[n] = {b} union ({d,e} - empty) = {b,d,e} (use Rule3)
out[p] = {b,d,e} (use Rule1)
in[p] = {b,d,e} (use Rule1)



转换为集合语言表述：
* $$in[n] = use[n] union (out[n] - def[n])$$
* $$out[n] = Union_{s \in succ[n]} in[s]$$



使用use-def链来实现整个流程的in-out分析





静态是在CFG上的 动态是在执行中的












