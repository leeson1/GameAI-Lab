# GameAI-Lab

GameAI-Lab 是一个面向游戏开发者的 AI 决策学习项目。

这个项目的目标不是直接研究强化学习论文，而是通过一个个可运行的小游戏 Demo，把 FSM、行为树、Utility AI、MDP、Q-Learning、DQN 等概念落到代码上。

核心原则：

> 每一个 AI 概念，都必须能对应到一段可运行的 C++ 代码。

---

## 1. 项目定位

本项目面向有游戏开发经验、但数学基础一般的工程师。

重点不是先推公式，而是先理解：

- 为什么需要这种 AI 决策方式？
- 它解决了普通 `if/else` 的什么问题？
- 它在游戏场景里应该怎么建模？
- 它最终如何写成 C++ 代码？

项目会优先从简单游戏场景出发，例如农场、自动战斗、塔防、挂机收益等。

---

## 2. 学习路线

建议路线如下：

```text
FSM
  ↓
Behavior Tree
  ↓
Utility AI
  ↓
MDP
  ↓
Value Iteration
  ↓
Q-Learning
  ↓
DQN
  ↓
Actor-Critic / PPO
```

每个阶段都要求满足：

1. 有清晰的游戏场景。
2. 有明确的 State / Action / Reward 或评分规则。
3. 有可运行代码。
4. 有实验输出。
5. 有文档解释算法与代码的对应关系。

---

## 3. 第一阶段 Demo：Farm AI

第一个 Demo 使用农场游戏作为实验场景。

原因：

- 状态空间小，容易枚举。
- 动作语义清晰，便于理解。
- 可以自然引入 Reward、Transition、Policy、Q Value。
- 不需要复杂战斗系统，也不依赖图形界面。

### 3.1 农场状态 State

初始设计：

```text
Empty    空地
Growing  生长中
Watered  已浇水
Mature   成熟
Dead     枯萎
```

### 3.2 农场动作 Action

初始设计：

```text
Plant    种植
Water    浇水
Wait     等待
Harvest  收获
Clear    清理枯萎作物
```

### 3.3 奖励 Reward

初始设计：

```text
Plant    -2    种植消耗
Water    -1    浇水消耗
Wait      0    等待本身没有立即收益
Harvest +10    收获收益
Clear    -1    清理成本
```

### 3.4 状态转移 Transition

初始设计：

```text
Empty + Plant
  -> 100% Growing, reward = -2

Growing + Water
  -> 100% Watered, reward = -1

Growing + Wait
  -> 40% Mature
  -> 20% Dead
  -> 40% Growing

Watered + Wait
  -> 80% Mature
  -> 5% Dead
  -> 15% Watered

Mature + Harvest
  -> 100% Empty, reward = +10

Dead + Clear
  -> 100% Empty, reward = -1
```

---

## 4. 第一版目录结构

项目按学习阶段组织，每个阶段包含自己的说明、代码和运行入口；跨阶段复用的环境与工具放在 `shared/`：

```text
GameAI-Lab/
├── README.md
├── stages/
│   ├── 01_fsm/
│   ├── 02_behavior_tree/
│   ├── 03_utility_ai/
│   ├── 04_mdp/
│   ├── 05_value_iteration/
│   ├── 06_q_learning/
│   ├── 07_dqn/
│   └── 08_actor_critic_ppo/
├── shared/
│   ├── common/
│   └── farm/
├── tests/
├── CMakeLists.txt
└── .gitignore
```

每个阶段内部后续统一使用以下结构：

```text
stages/xx_stage_name/
├── README.md
├── include/
├── src/
└── main.cpp
```

---

## 5. Agent 设计原则

项目会尽量把 Environment 和 Agent 分开。

### 5.1 Environment

Environment 负责描述游戏世界：

- 当前状态是什么？
- 某个动作是否合法？
- 执行动作后会得到什么奖励？
- 执行动作后会进入什么新状态？

伪代码：

```cpp
struct StepResult {
    State nextState;
    double reward;
    bool done;
};

class Environment {
public:
    State GetState() const;
    std::vector<Action> GetAvailableActions(State state) const;
    StepResult Step(Action action);
};
```

### 5.2 Agent

Agent 负责决策：

- FSM Agent：根据固定规则选择动作。
- Utility AI Agent：计算每个动作的人工评分。
- MDP Agent：基于已知转移模型求解最优 Policy。
- Q-Learning Agent：通过反复试错学习 Q Value。

伪代码：

```cpp
class Agent {
public:
    virtual Action ChooseAction(State state) = 0;
};
```

---

## 6. Utility AI 与 MDP 的核心区别

### Utility AI

Utility AI 的本质是人为设计评分函数：

```cpp
score = f(state, action);
```

例如：

```text
PlantScore = 80
WaterScore = 60
WaitScore = 30
```

分数只是排序依据，没有严格数学含义。

### MDP

MDP 的本质是定义环境模型：

```text
State
Action
Reward
Transition Probability
```

然后通过算法计算长期期望收益：

```text
Q(s, a) = Expected Future Return
```

也就是：

```text
当前奖励 + 未来累计奖励的期望
```

区别总结：

| 项目 | Utility AI | MDP |
|---|---|---|
| 本质 | 决策方法 | 数学建模方法 |
| 分数来源 | 人为设计 | 根据 Reward + Transition 推导 |
| 关注点 | 当前动作看起来值多少分 | 当前动作未来长期期望收益 |
| 是否需要转移概率 | 不需要 | 需要 |
| 输出 | 最优动作 | Value / Q Value / Policy |

---

## 7. 第一阶段目标

### Stage 1：Farm Utility AI

目标：

- 实现一个农场环境。
- 手写 Utility Score。
- 每一步选择当前分数最高的动作。

输出示例：

```text
State: Growing
Action Scores:
  Water = 80
  Wait  = 30
Choose: Water
```

### Stage 2：Farm MDP + Value Iteration

目标：

- 显式定义 Transition。
- 使用 Value Iteration 求解每个状态的最优 Value。
- 输出 Policy。

输出示例：

```text
Optimal Policy:
Empty   -> Plant
Growing -> Water
Watered -> Wait
Mature  -> Harvest
Dead    -> Clear
```

### Stage 3：Farm Q-Learning

目标：

- 不再直接使用完整 Transition 表求解。
- 通过反复与 Environment 交互学习 Q Value。
- 对比 Q-Learning 和 MDP Value Iteration 的结果。

输出示例：

```text
Episode 1000
Q(Growing, Water) = 6.41
Q(Growing, Wait)  = 4.28
Choose: Water
```

---

## 8. 后续扩展方向

Farm Demo 完成后，可以继续扩展：

```text
Battle AI       自动战斗 / 技能释放 / 血量管理
Idle Game AI    挂机收益最大化
Tower Defense   建塔 / 升级 / 卖塔
Card Game AI    出牌策略 / 长期收益
```

每个 Demo 都尽量复用同一套 Agent 接口。

---

## 9. 开发约定

初始约定：

- 语言：C++17
- 构建：CMake
- 第一阶段不引入第三方库
- 优先保证代码简单、可读、可运行
- 每个算法都要有独立 main 或示例入口
- 文档和代码同步更新

---

## 10. 当前任务列表

- [x] 创建基础目录结构
- [ ] 创建 CMakeLists.txt
- [ ] 实现 FarmEnvironment
- [ ] 实现 UtilityAIAgent
- [ ] 实现 MDPAgent + Value Iteration
- [ ] 实现 QLearningAgent
- [ ] 编写对应文档
- [ ] 对比 Utility AI / MDP / Q-Learning 的行为差异

---

## 11. 学习目标

完成第一轮 Farm Demo 后，应能清楚解释：

- Utility AI 为什么只是人工评分？
- Reward 和 Utility Score 有什么区别？
- Reward 和 Q Value 有什么区别？
- MDP 为什么需要 State Transition？
- Value Iteration 为什么需要多轮迭代？
- Policy 是什么时候生成的？
- Q-Learning 为什么可以在未知模型下学习？

最终目标：

> 不是背公式，而是能把 AI 决策思想写进游戏代码里。
