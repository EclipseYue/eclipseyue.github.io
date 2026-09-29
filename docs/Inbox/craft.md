# craft

## 文档计划

要写文档到docs/mine文件夹中

整理进度表文档
整体的编译链
基本数据的传输
分布式训练的调用路径
编译器的路径
profiler的功能与路径
runtime synapse的调用部分 ABI接口等
dlc设备的使用
TPU硬件的HBM layout方式

欠
MoE
fls attn
kimi MoBA MuonClip Kimi Linear Agent Swarm
kimi tech report
dpsk tech report MSA etc.
qwen tech report

## 待办

1. 恢复 DLC Runtime/driver 的 kernel completion。
2. 部署并确认加载 workspace DLC Custom Kernel catalog。
3. 对已修复项逐个执行 native syntest 和 public CPU differential。
4. 将通过项从 blocked promotion 为 verified。
5. 补齐确实不存在的 kernel，例如 avg_pool3d backward。
6. 最后重新运行完整 test_tpu 并取得明确的 pass/fail 集合。

## PyTorch 2.12：c10::DeviceAllocator

PyTorch 2.12 把这些能力向通用加速器抽象提升，增加了 `c10::DeviceAllocator`，见 `c10/core/CachingDeviceAllocator.h:211`：

```cpp
struct DeviceAllocator : public c10::Allocator {
    virtual bool initialized() = 0;
    virtual void emptyCache(MempoolId_t mempool_id = {0, 0}) = 0;
    virtual void recordStream(const DataPtr&, c10::Stream) = 0;
    virtual DeviceStats getDeviceStats(DeviceIndex) = 0;
    virtual void resetAccumulatedStats(DeviceIndex) = 0;
    virtual void resetPeakStats(DeviceIndex) = 0;
    virtual std::pair<size_t, size_t> getMemoryInfo(DeviceIndex);
};
```

这意味着 2.12 更希望各加速器后端提供统一的：

- caching allocator
- stream-safe 延迟释放
- memory stats
- empty_cache
- peak/reset
- memory snapshot
- graph/private mempool
- free/total memory 查询

但这仍然只是 allocator 管理能力的变化，不会改变普通 tensor 的逐元素布局。
