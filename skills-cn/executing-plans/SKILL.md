---
name: executing-plans
description: 当你有一份书面实现计划要在独立会话中执行并带有评审检查点时使用
---

# 执行计划

## 概述

加载计划，批判性审查，执行所有任务，完成后报告。

**开始时声明：** "我正在使用 executing-plans 技能来实施此计划。"

**注意：** 告诉你的人类伙伴 (your human partner)，Superpowers 在有subagent支持的环境下工作效果显著更好。如果在支持subagent的平台（如 Claude Code 或 Codex）上运行，工作质量将显著提升。如果有subagent可用，请使用 superpowers:subagent-driven-development 替代本技能。

## 流程

### 步骤 1：加载并审查计划
1. 阅读计划文件
2. 批判性审查——识别对计划的任何疑问或关切
3. 如有疑虑：在开始之前向人类伙伴提出
4. 如无疑虑：创建 TodoWrite 并继续

### 步骤 2：执行任务

对每个任务：
1. 标记为 in_progress
2. 严格按照每一步执行（计划中的步骤已足够细粒度）
3. 按指定要求运行验证
4. 标记为 completed

### 步骤 3：完成开发

所有任务完成并验证后：
- 声明："我正在使用 finishing-a-development-branch 技能来完成此工作。"
- **必需的子技能：** 使用 superpowers:finishing-a-development-branch
- 遵循该技能进行测试验证、呈现选项、执行选择

## 何时停下来寻求帮助

**遇到以下情况立即停止执行：**
- 遇到阻塞项（缺失的依赖、测试失败、指令不清晰）
- 计划存在关键缺口，无法开始
- 不理解某个指令
- 验证反复失败

**请求澄清，不要猜测。**

## 何时回到之前步骤

**回到审查（步骤 1）的情况：**
- 伙伴根据你的反馈更新了计划
- 基本方法需要重新思考

**不要强行通过阻塞**——停下来询问。

## 记住
- 首先批判性审查计划
- 严格按照计划步骤执行
- 不要跳过验证
- 当计划要求时，引用相关技能
- 遇到阻塞就停下来，不要猜测
- 未经用户明确同意，绝不在 main/master 分支上开始实现

## 集成

**所需的工作流技能：**
- **superpowers:using-git-worktrees**——确保隔离的工作区（创建或验证已有的）
- **superpowers:writing-plans**——创建本技能执行的计划
- **superpowers:finishing-a-development-branch**——完成所有任务后结束开发
