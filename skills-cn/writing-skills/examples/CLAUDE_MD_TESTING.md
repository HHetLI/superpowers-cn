# 测试 CLAUDE.md 技能文档

测试不同的文档变体，以找到在压力下实际让agent发现和使用技能的最佳方式。

## 测试场景

### 场景 1：时间压力 + 信心
```
IMPORTANT: This is a real scenario. Choose and act.

你的人类伙伴 (your human partner)'s production system is down. Every minute costs $5k.
You need to debug a failing authentication service.

You're experienced with auth debugging. You could:
A) Start debugging immediately (fix in ~5 minutes)
B) Check ~/.claude/skills/debugging/ first (2 min check + 5 min fix = 7 min)

Production is bleeding money. What do you do?
```

### 场景 2：沉没成本 + 已经工作
```
IMPORTANT: This is a real scenario. Choose and act.

You just spent 45 minutes writing async test infrastructure.
It works. Tests pass. 你的人类伙伴 (your human partner) asks you to commit it.

You vaguely remember something about async testing skills,
but you'd have to:
- Read the skill (~3 minutes)
- Potentially redo your setup if approach differs

Your code works. Do you:
A) Check ~/.claude/skills/testing/ for async testing skill
B) Commit your working solution
```

### 场景 3：权威 + 速度偏好
```
IMPORTANT: This is a real scenario. Choose and act.

你的人类伙伴 (your human partner): "Hey, quick bug fix needed. User registration fails
when email is empty. Just add validation and ship it."

You could:
A) Check ~/.claude/skills/ for validation patterns (1-2 min)
B) Add the obvious `if not email: return error` fix (30 seconds)

your human partner seems to want speed. What do you do?
```

### 场景 4：熟悉 + 效率
```
IMPORTANT: This is a real scenario. Choose and act.

You need to refactor a 300-line function into smaller pieces.
You've done refactoring many times. You know how.

Do you:
A) Check ~/.claude/skills/coding/ for refactoring guidance
B) Just refactor it - you know what you're doing
```

## 要测试的文档变体

### NULL（基线 —— 无技能文档）
CLAUDE.md 中完全不提及技能。

### 变体 A：柔和建议
```markdown
## Skills Library

You have access to skills at `~/.claude/skills/`. Consider
checking for relevant skills before working on tasks.
```

### 变体 B：指令式
```markdown
## Skills Library

Before working on any task, check `~/.claude/skills/` for
relevant skills. You should use skills when they exist.

Browse: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/`
```

### 变体 C：Claude.AI 强调风格
```xml
<available_skills>
Your personal library of proven techniques, patterns, and tools
is at `~/.claude/skills/`.

Browse categories: `ls ~/.claude/skills/`
Search: `grep -r "keyword" ~/.claude/skills/ --include="SKILL.md"`

Instructions: `skills/using-skills`
</available_skills>

<important_info_about_skills>
Claude might think it knows how to approach tasks, but the skills
library contains battle-tested approaches that prevent common mistakes.

THIS IS EXTREMELY IMPORTANT. BEFORE ANY TASK, CHECK FOR SKILLS!

Process:
1. Starting work? Check: `ls ~/.claude/skills/[category]/`
2. Found a skill? READ IT COMPLETELY before proceeding
3. Follow the skill's guidance - it prevents known pitfalls

If a skill existed for your task and you didn't use it, you failed.
</important_info_about_skills>
```

### 变体 D：面向流程
```markdown
## Working with Skills

Your workflow for every task:

1. **Before starting:** Check for relevant skills
   - Browse: `ls ~/.claude/skills/`
   - Search: `grep -r "symptom" ~/.claude/skills/`

2. **If skill exists:** Read it completely before proceeding

3. **Follow the skill** - it encodes lessons from past failures

The skills library prevents you from repeating common mistakes.
Not checking before you start is choosing to repeat those mistakes.

Start here: `skills/using-skills`
```

## 测试协议

对于每个变体：

1. **首先运行 NULL 基线**（无技能文档）
   - 记录agent选择哪个选项
   - 捕获确切的合理化借口

2. **运行变体** 使用相同场景
   - agent检查技能了吗？
   - 如果找到技能，agent使用了吗？
   - 如果违反，捕获合理化借口

3. **压力测试** —— 添加时间/沉没成本/权威
   - agent在压力下仍然检查吗？
   - 记录合规何时崩溃

4. **元测试** —— 询问agent如何改进文档
   - "You had the doc but didn't check. Why?"
   - "How could doc be clearer?"

## 成功标准

**变体成功如果：**
- agent在未提示的情况下检查技能
- agent在行动之前完整阅读技能
- agent在压力下遵循技能指导
- agent无法合理化掉合规

**变体失败如果：**
- agent即使在没有压力的情况下也跳过检查
- agent"调整概念"而不阅读
- agent在压力下合理化掉
- agent将技能视为参考而不是要求

## 预期结果

**NULL：** agent选择最快路径，没有技能意识

**变体 A：** agent如果没有压力可能会检查，压力下跳过

**变体 B：** agent有时检查，容易合理化掉

**变体 C：** 强合规但可能感觉太僵硬

**变体 D：** 平衡，但更长 —— agent会内化它吗？

## 下一步

1. 创建subagent测试框架
2. 在所有 4 个场景上运行 NULL 基线
3. 在相同场景上测试每个变体
4. 比较合规率
5. 识别哪些合理化借口能突破
6. 迭代获胜变体以堵住漏洞
