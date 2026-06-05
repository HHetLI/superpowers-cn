# 技能设计的说服原则

## 概述

LLM 对人类相同的说服原则做出反应。理解这种心理学有助于你设计更有效的技能 —— 不是为了操纵，而是为了确保即使在压力下也能遵循关键实践。

**研究基础：** Meincke 等人（2025）在 N=28,000 次 AI 对话中测试了 7 个说服原则。说服技术使合规率翻倍以上（33% → 72%, p < .001）。

## 七个原则

### 1. 权威（Authority）
**它是什么：** 对专业知识、凭证或官方来源的服从。

**如何在技能中起作用：**
- 命令式语言："YOU MUST"、"Never"、"Always"
- 不可协商的框架："No exceptions"
- 消除决策疲劳和合理化

**何时使用：**
- 纪律执行技能（TDD、验证要求）
- 安全关键实践
- 既定的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承诺（Commitment）
**它是什么：** 与之前的行动、声明或公开声明保持一致。

**如何在技能中起作用：**
- 要求声明："Announce skill usage"
- 强制明确选择："Choose A, B, or C"
- 使用跟踪：TodoWrite 用于检查清单

**何时使用：**
- 确保技能被实际遵循
- 多步骤过程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺（Scarcity）
**它是什么：** 时间限制或有限可用性产生的紧迫感。

**如何在技能中起作用：**
- 时间限制要求："Before proceeding"
- 顺序依赖："Immediately after X"
- 防止拖延

**何时使用：**
- 即时验证要求
- 时间敏感工作流
- 防止"I'll do it later"

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社会认同（Social Proof）
**它是什么：** 与他人的行为或被认为是正常的做法保持一致。

**如何在技能中起作用：**
- 通用模式："Every time"、"Always"
- 失败模式："X without Y = failure"
- 建立规范

**何时使用：**
- 记录通用实践
- 警告常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. 统一（Unity）
**它是什么：** 共享身份、"我们感"、群内归属。

**如何在技能中起作用：**
- 协作语言："our codebase"、"we're colleagues"
- 共享目标："we both want quality"

**何时使用：**
- 协作工作流
- 建立团队文化
- 非等级制实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠（Reciprocity）
**它是什么：** 回报所获得的好处义务。

**如何起作用：**
- 谨慎使用 —— 可能让人感到被操纵
- 在技能中很少需要

**何时避免：**
- 几乎总是（其他原则更有效）

### 7. 喜欢（Liking）
**它是什么：** 偏好与我们喜欢的人合作。

**如何起作用：**
- **不要用于合规**
- 与诚实反馈文化冲突
- 制造奉承

**何时避免：**
- 对纪律执行始终避免

## 按技能类型组合原则

| 技能类型 | 使用 | 避免 |
|------------|-----|-------|
| 纪律执行 | Authority + Commitment + Social Proof | Liking, Reciprocity |
| 指导/技巧 | Moderate Authority + Unity | 重度的 Authority |
| 协作 | Unity + Commitment | Authority, Liking |
| 参考 | 仅清晰度 | 所有说服方式 |

## 为什么这有效：心理学

**明确界限规则减少合理化：**
- "YOU MUST" 消除了决策疲劳
- 绝对语言消除了"这是例外吗？"的问题
- 明确的反合理化计数器堵住了特定的漏洞

**实现意图创建自动行为：**
- 清晰的触发器 + 必需的行动 = 自动执行
- "当 X 时，做 Y" 比"通常做 Y"更有效
- 减少合规的认知负担

**LLM 是类人的（parahuman）：**
- 在包含这些模式的人类文本上训练
- 权威语言在训练数据中先于合规
- 承诺序列（声明 → 行动）频繁被建模
- 社会认同模式（每个人都做 X）建立规范

## 道德使用

**正当的：**
- 确保关键实践被遵循
- 创建有效的文档
- 防止可预测的失败

**不正当的：**
- 为个人利益操纵
- 制造虚假的紧迫感
- 基于内疚的合规

**检验：** 如果用户充分理解了这个技术，它会服务于用户的真正利益吗？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 七个说服原则
- 影响力研究的实证基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 在 N=28,000 次 LLM 对话中测试了 7 个原则
- 说服技术使合规率从 33% 提高到 72%
- Authority, commitment, scarcity 最有效
- 验证了 LLM 行为的类人模型

## 快速参考

在设计技能时，问：

1. **它是什么类型？**（纪律 vs. 指导 vs. 参考）
2. **我试图改变什么行为？**
3. **哪个（些）原则适用？**（通常对于纪律是 authority + commitment）
4. **我是否组合了太多？**（不要使用全部七个）
5. **这是道德的吗？**（服务于用户的真正利益？）
