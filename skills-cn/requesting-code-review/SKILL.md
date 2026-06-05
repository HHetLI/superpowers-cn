---
name: requesting-code-review
description: 在完成任务、实现主要功能或合并之前使用，以验证工作是否满足需求
---

# 请求代码评审（Requesting Code Review）

派遣代码评审subagent，在问题扩散之前捕获它们。评审者获得精确构造的评估上下文——不会获得你的会话历史。这使评审者专注于工作成果而非你的思维过程，同时保留你自己的上下文以便继续工作。

**核心原则：** 尽早评审，经常评审。

## 何时请求评审

**强制：**
- 在subagent驱动开发（subagent-driven development）中每完成一个任务后
- 在完成主要功能后
- 在合并到 main 之前

**可选但有价值：**
- 卡住时（获得新视角）
- 重构前（基线检查）
- 修复复杂 bug 后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣代码评审subagent：**

使用 Task 工具的 `general-purpose` 类型，填入 `code-reviewer.md` 的模板

**占位符：**
- `{DESCRIPTION}` —— 你构建了什么，简要摘要
- `{PLAN_OR_REQUIREMENTS}` —— 它应该做什么
- `{BASE_SHA}` —— 起始提交
- `{HEAD_SHA}` —— 结束提交

**3. 根据反馈行动：**
- 立即修复 Critical（严重）问题
- 在继续之前修复 Important（重要）问题
- 记录 Minor（次要）问题以便后续处理
- 如果评审者错了，进行反驳（附理由）

## 示例

```
[刚完成 Task 2: 添加验证函数]

你: 让我在继续之前请求代码评审。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派遣代码评审subagent]
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，支持 4 种问题类型
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[subagent返回]:
  Strengths: 架构清晰，有真实测试
  Issues:
    Important: 缺少进度指示器
    Minor: 报告间隔使用魔法数字（100）
  Assessment: 可以继续

你: [修复进度指示器]
[继续 Task 3]
```

## 与工作流的集成

**subagent驱动开发（Subagent-Driven Development）：**
- 每个任务后评审
- 在问题积累之前捕获
- 修复后再进入下一个任务

**执行计划（Executing Plans）：**
- 每个任务后或在自然检查点评审
- 获取反馈，应用，继续

**临时开发（Ad-Hoc Development）：**
- 合并前评审
- 卡住时评审

## 红旗信号 (Red Flags)

**绝不：**
- 因为"很简单"而跳过评审
- 忽略 Critical（严重）问题
- 在 Important（重要）问题未修复时继续
- 与有效的技术反馈争辩

**如果评审者错了：**
- 用技术推理反驳
- 展示证明其有效的代码/测试
- 请求澄清

模板参见：requesting-code-review/code-reviewer.md
