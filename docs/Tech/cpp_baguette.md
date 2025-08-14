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

### Q: C++11新特性

八股：右值引用等等，C++11新特性，&和&&，static 和 const，多态



http协议：
客户端                      | 服务器
<方法> <请求方式> <请求路径>  | <HTTP版本><状态码><状态描述>
<请求头>                  | <响应头>
<请求体>                  | <响应体>


 