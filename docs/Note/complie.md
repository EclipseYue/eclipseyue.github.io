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

![截屏2025-04-24 15.17.53](https://tvax1.sinaimg.cn/large/008wagwxly1i0rx9nu0zcj31ow0l4djb.jpg)

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

先留着



## Chapter 6

Activation Records



















