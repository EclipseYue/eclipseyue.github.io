# AI Agent

AI Agent 记录“如何把模型组织成能完成任务的系统”。它不只是一次问答，而是包括规划、工具调用、检索、记忆、执行、评测和反馈的完整工作流。

## 目录

- [基础概念](fundamentals.md)：Agent、Workflow、Tool Use、Planning 的边界。
- [RAG](rag.md)：检索增强生成，适合知识密集型任务。
- [工具调用与 MCP](tool_use_mcp.md)：模型如何调用外部能力。
- [记忆与上下文工程](memory_context.md)：短期上下文、长期记忆和 prompt 组织。
- [多 Agent 协作](multi_agent.md)：角色分工、调度、协作和冲突处理。
- [工程化与评测](engineering_evaluation.md)：如何让 Agent 稳定上线。

## 学习路线

1. 先区分 Workflow 和 Agent：固定流程不等于自主智能体。
2. 学会 RAG：检索、重排、生成、引用和评测。
3. 加入工具调用：让模型能读写系统、调用 API、执行任务。
4. 加入记忆和状态：让系统能跨步骤保持上下文。
5. 最后做评测和观测：让 Agent 的失败可复现、可定位、可改进。
