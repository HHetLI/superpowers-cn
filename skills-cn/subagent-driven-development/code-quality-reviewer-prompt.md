# 代码质量评审者提示模板（Code Quality Reviewer Prompt Template）

在派遣代码质量评审者subagent时使用此模板。

**用途：** 验证实现是否构建良好（干净、有测试、可维护）

**仅在规格合规评审通过后派遣。**

```
Task tool (general-purpose):
  Use template at requesting-code-review/code-reviewer.md

  DESCRIPTION: [任务摘要，来自实现者的报告]
  PLAN_OR_REQUIREMENTS: Task N from [计划文件]
  BASE_SHA: [任务前的提交]
  HEAD_SHA: [当前提交]
```

**除了标准的代码质量关注点外，评审者还应检查：**
- 每个文件是否有一个明确的职责和一个定义良好的接口？
- 单元是否被分解为可以独立理解和测试的形式？
- 实现是否遵循计划中的文件结构？
- 此实现是否创建了已经很大的新文件，或显著增长了现有文件？（不要标记已有的文件大小问题——关注此更改造成的影响。）

**代码评审者返回：** 优点（Strengths）、问题（Issues：Critical/Important/Minor）、评估（Assessment）
