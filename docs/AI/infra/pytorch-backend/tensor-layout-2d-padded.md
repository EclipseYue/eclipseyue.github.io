# 2D Padded Layout

PyTorch 的逻辑 tensor 和设备侧真实 storage 不一定是一回事。对 AI 加速器后端来说，设备通常会要求按硬件访存粒度对齐，因此需要在 storage desc 中记录额外的物理布局信息。

## 逻辑视角

PyTorch tensor 主要由这些信息描述：

- `sizes`
- `strides`
- `storage_offset`
- dtype
- device

这套信息描述的是用户看到的逻辑张量。比如一个 shape 为 `[7, 4]` 的 `float32` tensor，逻辑上就是 28 个元素。

## 设备 storage 视角

设备侧可能还需要保存额外 desc：

```text
base_sizes       # PyTorch 逻辑 shape
base_strides     # PyTorch 逻辑 stride
storage_sizes    # 设备物理 storage 视角 shape
device_layout    # contiguous / 2d_padded / other
padded_dim0      # dim0 按硬件粒度对齐后的长度
```

2D padded 的核心是：设备不一定按 `numel * itemsize` 线性理解 tensor，而是把 tensor 压成二维视角，让最低维按对齐粒度补齐。

## 简化公式

假设设备要求 128 bytes 对齐，dtype 为 `float32`，每个元素 4 bytes：

```text
align_elems = 128 / itemsize = 32
padded_dim0 = round_up(sizes[0], align_elems)
physical_bytes = padded_dim0 * product(sizes[1:]) * itemsize
```

逻辑 shape 是 `[7, 4]` 时：

```text
logical_elements = 7 * 4 = 28
padded_dim0 = 32
physical_elements = 32 * 4 = 128
```

也就是说，逻辑数据只占其中一部分，剩余位置是 padding。

## 为什么这件事重要

layout 信息会影响：

- allocator 需要分配多少设备内存。
- H2D / D2H copy 是否能按逻辑元素直接拷贝。
- kernel 读取 tensor 时是否需要跳过 padding。
- contiguous / non-contiguous 判断是否仍然成立。
- CPU reference 和设备输出比较时是否需要做 layout-aware 处理。

如果只看 PyTorch 逻辑 shape，而忽略设备物理 storage，常见错误包括：

- 分配内存不足。
- D2H 回拷包含 padding 垃圾值。
- kernel 按连续线性访问导致越界或错位。
- 对齐后的 dim0 和原始 dim0 混用，导致 shape 边界错误。

## 迁移阶段的风险

在 P0 / P1 早期，代码中可能已经有字段、接口和检查路径，但物理 padding 行为尚未完整落成。这时要区分：

- desc 字段是否存在。
- storage sizes 是否真的按 padded dim0 写入。
- copy path 是否 layout-aware。
- kernel path 是否读取了正确的 physical shape。

因此调试 layout 问题时，不要只看“有没有 layout 字段”，还要看这个字段是否真正贯穿 allocator、copy、kernel 和 result check。

## 排查 checklist

- 逻辑 shape 和 physical storage shape 是否分开记录。
- dtype 改变时，alignment elements 是否同步改变。
- `storage_offset` 是否被正确处理。
- contiguous 判断是否区分逻辑 contiguous 和设备 layout contiguous。
- D2H 回拷时是否只比较有效逻辑区域。
- kernel 是否使用 padded dim0，而不是原始 dim0。
