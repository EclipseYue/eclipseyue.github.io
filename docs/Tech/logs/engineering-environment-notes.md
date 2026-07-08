# 工程环境经验

这页从工作 Obsidian 笔记中整理而来，只保留可公开复用的环境经验。原始笔记中的内网地址、账号、密码、token 和机器信息不迁入个人网站。

## root 与普通用户隔离

`root`、`sudo` 和普通用户的环境通常不是一套：

- `.ssh` 不同。
- Git config 不同。
- Conda / venv 不同。
- shell 启动脚本不同。
- 代理环境变量不同。

排查环境问题时先确认：

```bash
whoami
pwd
which python
python -V
git config --global user.name
git config --global user.email
env | sort
```

## source 与 bash 的区别

执行环境脚本时：

```bash
source ./env.sh
```

会修改当前 shell。

```bash
bash ./env.sh
```

只影响子 shell，脚本结束后当前终端环境不变。

代理、编译器路径、SDK 路径、Python 虚拟环境等问题经常卡在这里。

## Python / PyTorch 环境

本地开发 PyTorch fork 或设备插件时，`requirements.txt` 可能重新安装官方 `torch` 并覆盖本地 wheel。建议：

- 安装依赖前检查 requirements 是否包含 `torch`。
- 本地 wheel 安装后立刻确认 `torch.__version__`。
- vLLM 等项目如果需要本地 torch，安装依赖时要避免把 torch 覆盖掉。

常用检查：

```bash
python -c "import torch; print(torch.__version__); print(torch.__file__)"
python -m pip show torch
```

## Git 与 SSH

容器或新机器中先确认 SSH 和 Git 身份：

```bash
ssh -T git@github.com
git config --global user.name
git config --global user.email
```

大仓库 clone 出现连接中断时，先排查网络稳定性和 SSH keepalive。不要一开始就怀疑仓库损坏。

## 构建环境

C++ / PyTorch 后端开发常见依赖：

- `cmake`
- `ninja`
- `lld`
- `build-essential`
- Python headers / wheel 工具

经验：

- CMake 版本不够时，错误可能出现在很后面的构建阶段。
- Debug / Release / RelWithDebInfo 会明显影响构建速度和可调试性。
- 构建脚本中最好显式打印 Python、CMake、Ninja 和编译器路径。

## Runtime 调试

设备后端调试时，建议把问题拆成几层：

1. Python import 是否成功。
2. device count / set device 是否可用。
3. allocator 是否可用。
4. H2D / D2D / D2H memcpy 是否可用。
5. ATen op 是否注册到正确 dispatch key。
6. 参数打包是否正确。
7. runtime launch 是否真正发生。
8. 结果是否与 CPU reference 对齐。

不要直接从模型 failure 跳到 kernel。先压缩到最小 tensor、最小 op、最小 shape。

## 工作 SOP

一次工程任务建议按这个顺序收口：

1. 明确需求和验收命令。
2. 写最小公开行为测试。
3. 复现失败。
4. 做最小修复。
5. 跑本层测试和上层 smoke。
6. 解释数据流和调用链。
7. 写下可复用经验。
8. 提交。
