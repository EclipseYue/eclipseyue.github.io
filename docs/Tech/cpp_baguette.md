# C++ 八股

复习一下C++

### **Q**: Vector
C++ 中vector的底层实现？64位条件下vector容器本身的大小是多少，为什么？

**A**:
**大小:** vector由一个data指针(相当于数组头指针)、一个size和一个capacity组成。指针指向动态分配的内存，size表示当前元素个数，capacity表示分配的内存大小。64位条件下，vector容器本身的大小是24字节（8+8+8），因为指针占8字节，size和capacity各占8字节，总体为24 + capacity * sizeof(T)字节。

**结构:** vector实际是泛型的动态类型顺序表，因此底层是一段连续的内存空间，用三个指针指向内存空间,start,finish,end_of_storage
然后用他们来实现以下几种方法:begin(),end(),size(),capacity(),empty(),push_back(),pop_back()等。

**扩容:** 当 vector 的大小和容量相等（size==capacity）也就是满载时，如果再向其添加元素，那么 vector 就需要扩容。

vector 容器扩容的过程需要经历以下 3 步：
1. 完全弃用现有的内存空间，重新申请更大的内存空间（VS2015中以1.5倍扩容，GCC以2倍扩容。扩容倍数为2时，时间上占优势；扩容倍数为1.5时，空间上占优势。）
2. 将旧内存空间中的数据，按原有顺序移动到新的内存空间中；
3. 最后将旧的内存空间释放。


### **Q**: 多线程Vector
多线程下同时修改一个vector会有什么问题？如何解决？
**A**:在多线程环境下，如果多个线程同时修改同一个 `std::vector`，会遇到以下几个问题：

**数据竞态**
- **问题**：多个线程同时修改 `std::vector`（如插入、删除或修改元素）时，如果没有同步机制，可能会导致数据不一致或程序崩溃。例如，一个线程正在向 `vector` 添加元素，而另一个线程在同一时刻也在进行修改操作（如删除或修改元素），可能会导致访问非法内存或不一致的状态。
- **解决方法**：使用同步机制（如互斥锁 `std::mutex`）来保证在同一时刻只有一个线程访问 `vector`。

**内存访问冲突**
- **问题**：`std::vector` 内部是动态分配内存的，在进行扩容时，它可能会重新分配更大的内存区域，并将数据复制到新位置。如果有线程正在访问或修改 `vector`，而另一个线程执行扩容，可能导致内存访问冲突、指针悬挂或程序崩溃。
- **解决方法**：保证在对 `vector` 执行扩容时，其他线程不对其进行读写操作。可以通过锁保护整个 `vector` 或使用其他同步方式。

**不确定的迭代器行为**
- **问题**：如果一个线程正在迭代 `vector`，而另一个线程在同一时间修改它（例如，增加或删除元素），则迭代器可能变得无效，导致访问越界或未定义行为。
- **解决方法**：避免在多线程中同时进行迭代操作和修改操作。如果必须进行修改，使用锁或其他同步机制确保迭代和修改操作不冲突。

---

**解决方法：使用互斥锁**

使用 `std::mutex` 来保护对 `std::vector` 的访问，使得每次只能有一个线程操作 `vector`，其他线程必须等待。下面是一个使用 `std::mutex` 来同步访问 `vector` 的示例：

示例（C++）：
```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

std::vector<int> data;
std::mutex mtx;  // 互斥锁

// 修改vector的函数
void modifyVector(int value) {
    std::lock_guard<std::mutex> lock(mtx);  // 使用锁保护对vector的修改
    data.push_back(value);
}

// 读取vector的函数
void readVector() {
    std::lock_guard<std::mutex> lock(mtx);  // 使用锁保护对vector的读取
    for (int num : data) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::thread t1(modifyVector, 10);
    std::thread t2(modifyVector, 20);
    std::thread t3(readVector);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

**解释**：
- **`std::lock_guard<std::mutex>`**：这是一种 RAII 风格的锁，确保每次访问 `vector` 时都能自动获得锁，并且在操作完成后释放锁，避免手动管理锁。
- **同步访问**：在修改和读取 `vector` 时，所有线程都需要获取锁，保证同一时间只有一个线程可以访问 `vector`。

---

**解决方案总结**：

1. **使用 `std::mutex` 锁定 `vector`**：确保每次访问 `vector` 时，只有一个线程可以操作。
2. **避免在多线程中同时修改和遍历 `vector`**：操作和读取 `vector` 时，都要加锁，以防止数据不一致和内存冲突。
3. **如果需要频繁读写，考虑使用其他同步机制**：例如 **读写锁（`std::shared_mutex`）**，允许多个线程并行读取，而写入时仍然是独占访问。

这样通过使用同步机制可以有效地避免并发访问 `vector` 时的数据竞态问题，保证程序的正确性和稳定性。

### **Q**: C++11新特性
C++11有哪些重要的新特性？

**A**: 
**主要新特性包括:**

1. **auto关键字**: 自动类型推导
```cpp
auto x = 10;        // x为int类型
auto y = 3.14;      // y为double类型
auto z = "hello";   // z为const char*类型
```

2. **nullptr**: 空指针常量，替代NULL
```cpp
int* p = nullptr;   // 比NULL更安全
```

3. **范围for循环**: 简化容器遍历
```cpp
vector<int> v = {1, 2, 3, 4, 5};
for (const auto& x : v) {
    cout << x << " ";
}
```

4. **右值引用和移动语义**: 提高性能，减少拷贝
```cpp
string&& rref = move(str);  // 右值引用
```

5. **Lambda表达式**: 匿名函数
```cpp
auto lambda = [](int x) { return x * 2; };
```

6. **智能指针**: 自动内存管理
```cpp
unique_ptr<int> ptr = make_unique<int>(42);
shared_ptr<int> sptr = make_shared<int>(42);
```

7. **列表初始化**: 统一初始化语法
```cpp
vector<int> v{1, 2, 3, 4, 5};
int arr[]{1, 2, 3, 4, 5};
```

8. **decltype**: 类型推导
```cpp
int x = 10;
decltype(x) y = 20;  // y的类型为int
```

9. **constexpr**: 编译期常量表达式
```cpp
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
```

10. **线程支持库**: 标准多线程支持
```cpp
#include <thread>
std::thread t([]{ cout << "Hello from thread\n"; });
```

### **Q**: 引用(&)和右值引用(&&)
左值引用和右值引用的区别？什么是移动语义？

**A**:

**左值引用(&)**:
- 绑定到左值(有名字的对象，可以取地址)
- 延长对象生命周期
- 不能绑定到临时对象(右值)

```cpp
int x = 10;
int& lref = x;      // 正确，绑定到左值
// int& lref2 = 20; // 错误，不能绑定到右值
```

**右值引用(&&)**:
- 绑定到右值(临时对象，不能取地址)
- 支持移动语义，避免不必要的拷贝
- 可以"窃取"临时对象的资源

```cpp
int&& rref = 20;           // 正确，绑定到右值
string&& s = string("hello"); // 绑定到临时对象
```

**移动语义**:
移动构造函数和移动赋值运算符，"窃取"资源而不是拷贝

```cpp
class MyString {
    char* data;
    size_t size;
public:
    // 移动构造函数
    MyString(MyString&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;  // 窃取资源
        other.size = 0;
    }
    
    // 移动赋值运算符
    MyString& operator=(MyString&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

**完美转发**:
```cpp
template<typename T>
void wrapper(T&& arg) {
    func(std::forward<T>(arg));  // 保持参数的值类别
}
```

### **Q**: static关键字
static关键字的不同用法和作用？

**A**:

**1. 静态局部变量**:
- 只初始化一次，程序结束时销毁
- 保持函数调用间的状态

```cpp
int counter() {
    static int count = 0;  // 只初始化一次
    return ++count;
}
// 每次调用counter()，count都会递增
```

**2. 静态全局变量**:
- 限制在当前文件内可见
- 内部链接性

```cpp
static int global_var = 10;  // 只在当前文件可见
```

**3. 静态成员变量**:
- 属于类而不是对象
- 所有对象共享同一份
- 需要在类外定义

```cpp
class MyClass {
    static int static_var;  // 声明
public:
    static int getStaticVar() { return static_var; }
};
int MyClass::static_var = 0;  // 定义
```

**4. 静态成员函数**:
- 不依赖于对象实例
- 不能访问非静态成员
- 可以通过类名直接调用

```cpp
class Math {
public:
    static int add(int a, int b) {
        return a + b;
    }
};
// 调用: Math::add(1, 2)
```

### **Q**: const关键字
const的各种用法和作用？

**A**:

**1. 常量变量**:
```cpp
const int x = 10;        // x不能被修改
const int* p1 = &x;      // 指向常量的指针，不能通过p1修改值
int* const p2 = &y;      // 常量指针，p2不能指向其他地址
const int* const p3 = &x; // 常量指针指向常量
```

**2. 常量成员函数**:
- 不修改对象状态
- const对象只能调用const成员函数

```cpp
class MyClass {
    int value;
public:
    int getValue() const {   // const成员函数
        return value;        // 不能修改成员变量
    }
    
    void setValue(int v) {   // 非const成员函数
        value = v;
    }
};

const MyClass obj;
obj.getValue();  // 正确
// obj.setValue(10); // 错误，const对象不能调用非const函数
```

**3. 常量引用**:
```cpp
const int& ref = x;      // 常量引用，不能通过ref修改x
```

**4. 返回值const**:
```cpp
const string& getName() const {  // 返回常量引用
    return name;
}
```

**5. mutable关键字**:
允许在const函数中修改特定成员

```cpp
class Cache {
    mutable int cache_hits;  // 可在const函数中修改
public:
    int getValue() const {
        ++cache_hits;        // 正确，mutable成员
        return value;
    }
};
```

### **Q**: 多态
C++中的多态机制是什么？静态多态和动态多态的区别？

**A**:

**多态定义**: 同一接口，不同实现。允许使用基类指针或引用调用派生类的方法。

**静态多态(编译期多态)**:
- 函数重载
- 运算符重载  
- 模板

```cpp
// 函数重载
void print(int x) { cout << "int: " << x << endl; }
void print(double x) { cout << "double: " << x << endl; }

// 模板
template<typename T>
void func(T x) { cout << x << endl; }
```

**动态多态(运行期多态)**:
通过虚函数和继承实现

```cpp
class Animal {
public:
    virtual void makeSound() {  // 虚函数
        cout << "Animal sound" << endl;
    }
    virtual ~Animal() = default;  // 虚析构函数
};

class Dog : public Animal {
public:
    void makeSound() override {   // 重写虚函数
        cout << "Woof!" << endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        cout << "Meow!" << endl;
    }
};

// 使用多态
Animal* animals[] = {new Dog(), new Cat()};
for (auto* animal : animals) {
    animal->makeSound();  // 运行时决定调用哪个版本
}
```

**虚函数实现原理**:
- 每个包含虚函数的类都有虚函数表(vtable)
- 每个对象都有虚函数表指针(vptr)
- 通过vptr找到vtable，再找到对应的函数

**纯虚函数和抽象类**:
```cpp
class Shape {
public:
    virtual double getArea() = 0;  // 纯虚函数
    virtual ~Shape() = default;
};
// Shape是抽象类，不能实例化
```

**虚析构函数的重要性**:
```cpp
class Base {
public:
    virtual ~Base() {  // 虚析构函数
        cout << "Base destructor" << endl;
    }
};

class Derived : public Base {
public:
    ~Derived() {
        cout << "Derived destructor" << endl;
    }
};

Base* ptr = new Derived();
delete ptr;  // 正确调用Derived析构函数，再调用Base析构函数
```

### **Q**: HTTP协议
HTTP请求和响应的格式是什么？

**A**:

**HTTP请求格式**:
```
GET /index.html HTTP/1.1          // 请求行：方法 路径 版本
Host: www.example.com             // 请求头
User-Agent: Mozilla/5.0
Accept: text/html
Content-Length: 0

[请求体]                          // 可选，GET通常为空
```

**HTTP响应格式**:
```
HTTP/1.1 200 OK                   // 状态行：版本 状态码 状态描述
Content-Type: text/html           // 响应头
Content-Length: 1234
Server: Apache/2.4

<html>                            // 响应体
<body>Hello World</body>
</html>
```

**常见HTTP方法**:
- **GET**: 获取资源
- **POST**: 提交数据
- **PUT**: 更新资源
- **DELETE**: 删除资源
- **HEAD**: 获取头部信息
- **OPTIONS**: 获取支持的方法

**常见状态码**:
- **2xx**: 成功
  - 200 OK: 请求成功
  - 201 Created: 资源创建成功
- **3xx**: 重定向
  - 301 Moved Permanently: 永久重定向
  - 302 Found: 临时重定向
- **4xx**: 客户端错误
  - 400 Bad Request: 请求错误
  - 401 Unauthorized: 未授权
  - 404 Not Found: 资源未找到
- **5xx**: 服务器错误
  - 500 Internal Server Error: 服务器内部错误
  - 503 Service Unavailable: 服务不可用

**HTTP头部字段**:
```
// 请求头
Host: www.example.com             // 目标主机
User-Agent: browser-info          // 客户端信息
Accept: text/html                 // 可接受的内容类型
Cookie: session=abc123            // Cookie信息

// 响应头
Content-Type: text/html           // 内容类型
Content-Length: 1234              // 内容长度
Set-Cookie: session=xyz789        // 设置Cookie
Cache-Control: max-age=3600       // 缓存控制
```

### **Q**: C++锁机制
C++中有哪些锁类型？它们的区别和使用场景是什么？

**A**:

**1. 互斥锁(mutex)**:
最基本的锁，同一时间只能有一个线程获取

```cpp
#include <mutex>

std::mutex mtx;

void criticalSection() {
    mtx.lock();        // 手动加锁
    // 临界区代码
    mtx.unlock();      // 手动解锁
}

// 推荐使用RAII方式
void safeCriticalSection() {
    std::lock_guard<std::mutex> lock(mtx);  // 自动加锁
    // 临界区代码
}  // 自动解锁
```

**2. 递归锁(recursive_mutex)**:
允许同一线程多次获取同一个锁

```cpp
std::recursive_mutex rec_mtx;

void recursiveFunction(int n) {
    std::lock_guard<std::recursive_mutex> lock(rec_mtx);
    if (n > 0) {
        recursiveFunction(n - 1);  // 同一线程再次获取锁
    }
}
```

**3. 定时锁(timed_mutex)**:
支持超时的互斥锁

```cpp
std::timed_mutex tm_mtx;

void timedLockExample() {
    // 尝试在1秒内获取锁
    if (tm_mtx.try_lock_for(std::chrono::seconds(1))) {
        // 成功获取锁
        // 临界区代码
        tm_mtx.unlock();
    } else {
        // 超时，处理失败情况
    }
}
```

**4. 读写锁(shared_mutex, C++17)**:
允许多个线程同时读，但写操作独占

```cpp
#include <shared_mutex>

std::shared_mutex rw_mtx;
int shared_data = 0;

// 读操作
void reader() {
    std::shared_lock<std::shared_mutex> lock(rw_mtx);  // 共享锁
    int value = shared_data;  // 多个线程可以同时读
}

// 写操作
void writer() {
    std::unique_lock<std::shared_mutex> lock(rw_mtx);  // 独占锁
    shared_data = 42;  // 只有一个线程可以写
}
```

**5. 条件变量(condition_variable)**:
用于线程间同步，等待特定条件

```cpp
#include <condition_variable>

std::mutex cv_mtx;
std::condition_variable cv;
bool ready = false;

// 等待线程
void waiter() {
    std::unique_lock<std::mutex> lock(cv_mtx);
    cv.wait(lock, []{ return ready; });  // 等待ready为true
    // 条件满足后继续执行
}

// 通知线程
void notifier() {
    {
        std::lock_guard<std::mutex> lock(cv_mtx);
        ready = true;
    }
    cv.notify_one();  // 通知一个等待线程
    // cv.notify_all();  // 通知所有等待线程
}
```

**6. 自旋锁(atomic_flag)**:
轻量级锁，使用忙等待

```cpp
#include <atomic>

class SpinLock {
    std::atomic_flag flag = ATOMIC_FLAG_INIT;
public:
    void lock() {
        while (flag.test_and_set(std::memory_order_acquire)) {
            // 忙等待
        }
    }
    
    void unlock() {
        flag.clear(std::memory_order_release);
    }
};
```

**锁的RAII封装类**:

**lock_guard**: 最简单的RAII锁封装
```cpp
{
    std::lock_guard<std::mutex> lock(mtx);
    // 自动加锁，作用域结束自动解锁
}
```

**unique_lock**: 更灵活的锁封装
```cpp
std::unique_lock<std::mutex> lock(mtx);
// 可以手动解锁
lock.unlock();
// 可以重新加锁
lock.lock();
// 可以转移所有权
std::unique_lock<std::mutex> lock2 = std::move(lock);
```

**shared_lock**: 共享锁封装(C++14)
```cpp
std::shared_lock<std::shared_mutex> lock(rw_mtx);
// 允许多个线程同时持有
```

**死锁预防**:

**1. 避免嵌套锁**:
```cpp
// 危险：可能死锁
void thread1() {
    std::lock_guard<std::mutex> lock1(mtx1);
    std::lock_guard<std::mutex> lock2(mtx2);
}

void thread2() {
    std::lock_guard<std::mutex> lock1(mtx2);  // 顺序相反
    std::lock_guard<std::mutex> lock2(mtx1);
}
```

**2. 使用std::lock同时获取多个锁**:
```cpp
void safeLocking() {
    std::lock(mtx1, mtx2);  // 同时获取多个锁
    std::lock_guard<std::mutex> lock1(mtx1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(mtx2, std::adopt_lock);
}
```

**3. 使用scoped_lock(C++17)**:
```cpp
void scopedLocking() {
    std::scoped_lock lock(mtx1, mtx2);  // 自动处理多锁获取
}
```

**性能考虑**:

**锁的开销比较**:
- **atomic操作** < **自旋锁** < **互斥锁** < **读写锁**

**选择建议**:
- **短临界区**: 使用自旋锁或atomic
- **长临界区**: 使用互斥锁
- **读多写少**: 使用读写锁
- **需要超时**: 使用timed_mutex
- **递归调用**: 使用recursive_mutex

**最佳实践**:

```cpp
class ThreadSafeCounter {
    mutable std::mutex mtx;  // mutable允许在const函数中使用
    int count = 0;
    
public:
    void increment() {
        std::lock_guard<std::mutex> lock(mtx);
        ++count;
    }
    
    int getValue() const {
        std::lock_guard<std::mutex> lock(mtx);
        return count;
    }
    
    // 返回副本而不是引用，避免锁外访问
    int getValueSafe() const {
        std::lock_guard<std::mutex> lock(mtx);
        return count;  // 返回副本
    }
};
```

**无锁编程(Lock-Free)**:
使用atomic操作避免锁

```cpp
#include <atomic>

class LockFreeCounter {
    std::atomic<int> count{0};
    
public:
    void increment() {
        count.fetch_add(1, std::memory_order_relaxed);
    }
    
    int getValue() const {
        return count.load(std::memory_order_relaxed);
    }
};
```


