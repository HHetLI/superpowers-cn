---
name: writing-plans
description: 当你有多步骤任务的规格说明或需求时，在触碰代码之前使用
---

# 编写计划

## 概述

编写全面的实现计划，假设工程师对我们的代码库零上下文、品味可疑（questionable taste）。记录他们需要知道的一切：每个任务要修改哪些文件、代码、测试、可能需要查阅的文档、如何测试。将整个计划分解为小步骤（bite-sized）任务。DRY。YAGNI。TDD。频繁提交。

假设他们是有经验的开发者，但对我们的工具集或问题领域几乎一无所知。假设他们不太了解好的测试设计。

**开始时声明：** "我正在使用 writing-plans 技能来创建实现计划。"

**上下文：** 如果在隔离的 worktree 中工作，它应该已经通过 `superpowers:using-git-worktrees` 技能在执行时创建好了。

**保存计划到：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划位置的偏好覆盖此默认值）

## 范围检查

如果规范涵盖多个独立的子系统，它应该在 brainstorming 阶段被分解为子项目规范。如果没有，建议将其拆分为单独的计划 —— 每个子系统一个。每个计划应该自身产生可工作的、可测试的软件。

## 文件结构

在定义任务之前，规划哪些文件将被创建或修改，以及每个文件负责什么。这是分解决策被锁定的地方。

- 设计具有清晰边界和良好定义接口的单元。每个文件应该有一个清晰的职责。
- 你对可以一次性保留在上下文中的代码推理得最好，当文件专注时你的编辑也更可靠。偏好更小、更专注的文件，而不是做太多的大文件。
- 一起更改的文件应该一起存在。按职责拆分，而不是按技术层拆分。
- 在现有代码库中，遵循既定模式。如果代码库使用大文件，不要单方面重构 —— 但如果你正在修改的文件已经变得笨重，在计划中包括拆分是合理的。

此结构会影响任务分解。每个任务应该产生自包含的更改，这些更改独立地有意义。

## 小步骤（Bite-Sized）任务粒度

**每个步骤是一个操作（2-5 分钟）：**
- "Write the failing test" —— 步骤
- "Run it to make sure it fails" —— 步骤
- "Implement the minimal code to make the test pass" —— 步骤
- "Run the tests and make sure they pass" —— 步骤
- "Commit" —— 步骤

## 计划文档头部

**每个计划必须以这个头部开始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 没有占位符

每个步骤必须包含工程师需要的实际内容。以下是**计划失败** —— 永远不要写它们：
- "TBD"、"TODO"、"implement later"、"fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above"（没有实际测试代码）
- "Similar to Task N"（重复代码 —— 工程师可能乱序阅读任务）
- 描述要做什么但没有展示如何做的步骤（代码步骤需要代码块）
- 引用未在任何任务中定义的类型、函数或方法

## 记住
- 始终使用精确的文件路径
- 每个步骤中的完整代码 —— 如果步骤更改代码，展示代码
- 精确的命令以及预期输出
- DRY、YAGNI、TDD、频繁提交

## 自我审查

在编写完完整计划后，用新鲜的眼光看规范，并对照它检查计划。这是你自己运行的检查清单 —— 不是subagent分派。

**1. 规范覆盖度：** 浏览规范中的每个章节/需求。你能指出一个实现它的任务吗？列出任何差距。

**2. 占位符扫描：** 在计划中搜索 Red Flags —— 任何来自上面"没有占位符"部分的模式。修复它们。

**3. 类型一致性：** 你在后续任务中使用的类型、方法签名和属性名称是否与你在早期任务中定义的匹配？一个在任务 3 中调用的函数是 `clearLayers()`，但在任务 7 中是 `clearFullLayers()`，这就是一个 bug。

如果你发现问题，在行内修复它们。无需重新审查 —— 修复后继续。如果你发现规范需求没有对应任务，添加该任务。

## 执行交接

保存计划后，提供执行选择：

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**如果选择 Subagent-Driven：**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- 每个任务一个全新的subagent+ 两阶段审查

**如果选择 Inline Execution：**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- 批量执行，带检查点进行审查
