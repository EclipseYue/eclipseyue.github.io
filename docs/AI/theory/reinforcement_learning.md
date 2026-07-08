# 强化学习

强化学习研究智能体如何通过与环境交互学习策略。它不只关心单步预测是否正确，更关心长期回报和决策序列。

## 基础定义

- Agent：做决策的主体。
- Environment：与 Agent 交互的外部系统。
- State：环境状态。
- Action：Agent 可采取的动作。
- Reward：环境反馈的奖励。
- Policy：从状态到动作的决策规则。

强化学习常用马尔可夫决策过程表示：

$$
\mathcal{M}=(\mathcal{S}, \mathcal{A}, P, R, \gamma)
$$

其中 $P$ 是状态转移概率，$R$ 是奖励函数，$\gamma$ 是折扣因子。

## 方法地图

- 值函数方法：Q-Learning、DQN。
- 策略梯度：REINFORCE、Actor-Critic。
- 近端优化：PPO。
- 离线强化学习：从历史数据中学习策略。
- 模仿学习：从专家轨迹中学习行为。

## 与大模型的关系

- RLHF：用人类偏好训练奖励模型，再优化语言模型策略。
- RLAIF：用 AI 反馈替代或补充人类反馈。
- Agent 任务中的规划和工具使用，也可以用强化学习观点分析。

## 待补充

- Bellman 方程推导。
- Policy Gradient 推导。
- PPO 的 clipped objective。
