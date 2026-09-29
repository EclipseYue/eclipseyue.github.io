# pytorch后端解读

> 目标：理解一个 op 从 Python 调用到后端 kernel 的完整过程，以及 `eager` / `Graph` 两种执行模式下的 stream 与同步语义。

## 从简单计算到算子

- Python 层：`torch.add(a, b)` 走到 `Tensor` 方法或 ATen 函数绑定，实质是进入 `c10::Dispatcher`。
- schema：每个 op 都有声明式 schema（如 `aten::add.Tensor(Tensor self, Tensor other, Scalar alpha=1) -> Tensor`），它约定了重载解析、dtype 提升等契约。
- dispatcher：按 dispatch key 选中实现；`at::native` 里是参考实现，后端只需覆盖自己关心的 op，其余交给 fallback。
- OpPreparation：多数 op 不自己造输出，而是先算出输出的 shape / dtype / device（借助 `TensorIterator`、类型提升规则等），再用 `at::empty` 之类完成分配。
- structured kernel：把“元信息推理”与“计算”分离。后端只实现计算部分，元信息复用通用逻辑；也可注册 meta 函数来支持 `FakeTensor` / meta 设备推理。
- 后端最先要打通的三个基础 op：`aten::empty`、`aten::empty_strided`、`aten::_copy_from`——其余算子大多建立在它们之上。

```text
torch.add(a, b)
  -> Dispatcher（按 dispatch key 选实现）
  -> OpPreparation：计算输出 shape / dtype / device
  -> at::empty -> 后端 allocator 分配
  -> 后端 kernel 写入输出
```

!!! note "TODO"
    - 挑一个有代表性的算子（如 `add` 或 `softmax`），把 schema、meta 推理、后端实现逐层拆开记录。
    - 记录 dtype 提升与隐式类型转换发生在哪一层。

## 'Stream' 'Event'机制实现

- `c10::Stream`：轻量句柄，内含 `StreamId` + `DeviceIndex` + 设备类型，不持有资源，可自由拷贝。
- `c10::Event`：跨 stream 的完成标记，带 `EventFlag` 与状态查询 / 同步接口（如 `synchronize`、`query`）。
- 当前 stream 的查询与设置：CPU / CUDA 各有 `getCurrentCUDAStream()` 这类接口；自定义后端必须提供等价实现（社区常见命名形如 `getCurrentPrivateUse1Stream`，以所用版本为准），否则 guard 和 kernel 都不知道该往哪条流排队。
- guard：`c10::DeviceGuard` / `c10::OptionalDeviceGuard` / `impl::InlineDeviceGuard` 在作用域内切换“当前设备 + 当前 stream”，退出时还原。这是后端必须支持的接口，否则大量框架代码无法工作。
- 延迟释放：`recordStream` 把内存与 stream 绑定，保证该 stream 上后续工作完成前内存不被复用；跨 stream 使用时需显式 event 同步。
- 典型同步点：H2D 拷贝后要让计算流等待拷贝流；D2H 读回结果前要等待计算完成。同步粒度选错是“结果偶发错误”的高频原因。

```text
stream A: H2D copy --recordEvent--> event
stream B: compute  --waitEvent(event)--> 继续
```

!!! note "TODO"
    - 整理“当前 stream / 默认 stream / 自定义 stream”三者的行为差异。
    - 记录一次跨 stream 同步缺失导致错误结果的实际案例。

## 'eager' 'Graph'机制实现

- eager 模式：Python 调用即执行，autograd 在运行时动态记录反向图。灵活，但 Python 开销大、kernel launch 密集。
- 图捕获的前置条件：后端需要支持 stream 语义、allocator 的 `recordStream`、以及可被捕获的内存池（graph / private mempool），否则捕获会失败或产生悬垂指针。
- `torch.compile` 链路（概念层）：
  - Dynamo：把 Python 字节码转成 FX graph；遇到无法追踪的代码就产生 **graph break**，把图拆成多段。
  - AOTAutograd：为前向 / 反向生成带 autograd 信息的图，并做图变换。
  - Inductor：把图降级为可执行代码 / kernel（默认生成 Triton 或 C++）。
- `FakeTensor` / meta device：在“只跑元信息、不碰真实内存”的 fake 模式下完成 shape / dtype 推理与图变换，是编译链路能跑通的关键支撑。
- 后端接入点：提供自定义 compiler backend 并注册，把 Inductor 产出或自己的图编译结果接到设备 runtime。

!!! note "TODO"
    - 补 `torch.compile(backend=...)` 的最小自定义后端示例。
    - 记录 graph break 的实际触发点，以及它如何影响后端的编译粒度。
