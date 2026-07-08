# 简历复习与项目准备

这里放面试前快速复习用的项目表达。正式简历只保留精简版，细节放在这里按“项目概述 - 技术亮点 - 可追问点 - 改进方向”维护。

## 使用方式

- 每个项目准备 30 秒、1 分钟和 3 分钟三个版本。
- 技术亮点只保留自己能展开讲清楚的内容。
- 被问倒的问题当天补回对应项目。
- 如果某个细节变成通用知识，迁到 CS、AI 或工程技术目录。

## 项目分组

### AI / Infra / Agent

- 智能 AI 笔记本系统：本地 RAG、多模型接入、Agent 编排、编辑器联动。
- AI Infra 工作内容：PyTorch 后端、算子接入、runtime 调试、custom kernel、精度定位。
- AI 全栈实习：营销业务 AIGC 应用、Agent 后端、Spring AI、业务流程编排。

### 系统与底层

- 微型操作系统 MiniOS：进程调度、分页、缺页异常、中断和系统调用。
- RISC-V 五段流水线 CPU：RV32I、流水线冒险、forwarding、分支预测。
- 计算机网络协议栈：TCP/IP、滑动窗口、路由、Mininet、Wireshark。

### 应用与工程

- FitSeek 体重管理系统：Vue3、Uni-app、Node.js、MongoDB、Docker、CI/CD。
- 3D 迷宫逃杀：OpenGL、Prim、A*、光照、音效和状态恢复。
- 四键交互音游：Qt、MVVM、多线程、音符渲染和输入判定。
- 微型数据库 MiniSQL：B+ 树、Buffer Pool、LRU、索引和简单 SQL 引擎。
- 图书管理系统：Java、Vue3、MySQL、多用户并发和日志恢复。

## 工作经历表达

### AI Infra 工程方向

**一句话版本**

参与国产 AI 加速器软件栈相关工作，关注 PyTorch 后端、算子接入、custom kernel、runtime 调试和模型推理链路。

**可展开版本**

- 做 PyTorch 设备扩展和后端适配相关梳理，理解 PrivateUse1、ATen dispatch、参数打包、CPU/设备后端双路径验证和 fallback 机制。
- 参与 custom kernel 与 runtime 问题定位，使用模拟器、真实硬件、CPU Reference、硬件感知参考和模型现场 dump 做算子级复现。
- 沉淀工程知识库、case study 和调试流程，帮助新任务快速恢复领域上下文。

**可追问点**

- PyTorch 后端如何接入一个新设备。
- ATen 算子如何从 Python 调用走到后端实现。
- custom op 和 custom kernel 的边界。
- 为什么需要 CPU Reference 和 Hardware-Aware Reference。
- 模型级 failure 如何压缩到算子级复现。

### AI 全栈实习

**一句话版本**

参与营销业务 AI 应用研发，完成 Agent / AIGC 后端能力从需求拆解、接口设计到部署交付的闭环。

**可展开版本**

- 基于 Spring AI 和大模型接口封装业务能力，负责模型调用、流程编排、结果处理和服务接口。
- 与产品、运营、设计协作，把原型能力拆成可上线、可迭代的功能模块。
- 参与 AI 应用平台和技术中台建设，关注服务稳定性、复用性和后续扩展。

**可追问点**

- Agent 应用和普通 LLM API 封装的区别。
- 如何设计模型调用的错误处理和降级。
- 业务流程中如何做 prompt、工具调用和结果校验。
- AI 应用上线后如何评估效果。

## 项目表达模板

```text
项目背景：
我负责的部分：
核心难点：
技术方案：
验证方式：
结果和收益：
如果重做会怎么改：
```

## 面试复盘模板

```text
公司 / 岗位：
面试轮次：
被问到的问题：
回答不好的点：
需要补的知识：
下一次表达方式：
```
