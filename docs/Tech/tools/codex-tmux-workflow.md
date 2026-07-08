# Codex 与 tmux 工作流

这里整理本地使用 Codex 时的会话管理和长任务运行方式。重点是让任务可恢复、上下文可追踪，而不是依赖某一次终端窗口。

## tmux 基础

长任务建议放在 `tmux` 中运行：

```bash
tmux new -s codex
tmux attach -t codex
tmux kill-session -t codex
```

常见流程：

1. 新建 session。
2. 在 session 中启动 Codex 或其他长任务。
3. 需要离开时按 `Ctrl+b`，再按 `d` 分离。
4. 回来后用 `tmux attach -t codex` 恢复。

这样即使本地终端窗口关闭，远端 shell 仍可继续运行。

## Codex 会话整理

可以使用会话管理工具整理本地 Codex 历史：

```bash
npm install -g codex-session-manager
codex-session-manager
```

常用操作：

| 操作 | 快捷键 | 用途 |
| --- | --- | --- |
| 搜索 | `/` | 按名称或标签查找会话 |
| 重命名 | `r` | 给会话补清晰标题 |
| 编辑标签 | `t` | 添加任务类型、项目、阶段 |
| 归档 | `a` | 收起已结束会话 |
| 详情 | `d` | 查看会话预览 |
| 帮助 | `h` 或 `?` | 查看完整快捷键 |

建议给会话打两类标签：

- 项目标签：`site`、`pytorch-backend`、`paper`。
- 阶段标签：`debug`、`docs`、`review`、`commit`。

## 权限与安全

不要把“绕过审批和沙盒”的命令写进默认 alias，也不要长期放在 shell 启动脚本里。更稳妥的做法是：

- 默认使用普通安全模式。
- 只有明确需要修改系统级路径或联网安装依赖时，再单次授权。
- 执行前确认工作区无未保存的重要修改。
- 对 destructive 命令保持显式确认。

## 会话恢复原则

长上下文任务容易中断，恢复时先做三件事：

1. `git status` 确认工作区状态。
2. 查看最近一条用户需求，避免继续做旧任务。
3. 用简短 TODO 重新确认剩余步骤。

如果任务涉及代码或文档迁移，结束前必须：

- 跑格式或构建检查。
- 检查是否有敏感信息。
- 提交信息写清楚“做了什么”，不要只写 `update`。
