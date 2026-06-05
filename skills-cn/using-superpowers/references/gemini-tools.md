# Gemini CLI 工具映射

技能使用 Claude Code 的工具名称。当你在技能中遇到这些名称时，使用你平台的等价物：

| 技能引用 | Gemini CLI 等价物 |
|-----------------|----------------------|
| `Read`（文件读取） | `read_file` |
| `Write`（文件创建） | `write_file` |
| `Edit`（文件编辑） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名称搜索文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` 工具（调用技能） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（分派subagent） | `@agent-name`（参见[subagent支持](#subagent支持)） |

## subagent支持

Gemini CLI 通过 `@` 语法原生支持subagent。使用内置的 `@generalist` agent来分派任何任务 —— 它可以访问所有工具并遵循你提供的提示。

当技能说要分派一个命名agent类型时，使用 `@generalist` 配合技能提示模板中的完整提示：

| 技能指令 | Gemini CLI 等价物 |
|-------------------|----------------------|
| `Task tool (superpowers:implementer)` | `@generalist` 配合已填充的 `implementer-prompt.md` 模板 |
| `Task tool (superpowers:spec-reviewer)` | `@generalist` 配合已填充的 `spec-reviewer-prompt.md` 模板 |
| `Task tool (superpowers:code-reviewer)` | `@code-reviewer`（内置agent）或 `@generalist` 配合已填充的审查提示 |
| `Task tool (superpowers:code-quality-reviewer)` | `@generalist` 配合已填充的 `code-quality-reviewer-prompt.md` 模板 |
| `Task tool (general-purpose)` 配合内联提示 | `@generalist` 配合你的内联提示 |

### 提示填充

技能提供带有占位符的提示模板，如 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]`。填充所有占位符，并将完整提示作为消息传递给 `@generalist`。提示模板本身包含了agent的角色、审查标准和预期输出格式 —— `@generalist` 会遵循它。

### 并行分派

Gemini CLI 支持并行subagent分派。当技能要求你并行分派多个独立的subagent任务时，在同一提示中一起请求所有这些 `@generalist` 或命名subagent任务。保持依赖任务顺序执行，但不要仅仅为了保留更简单的历史记录而将独立的subagent任务串行化。

## 额外的 Gemini CLI 工具

这些工具在 Gemini CLI 中可用，但在 Claude Code 中没有等价物：

| 工具 | 用途 |
|------|---------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将事实持久化到跨会话的 GEMINI.md 中 |
| `ask_user` | 从用户请求结构化输入 |
| `tracker_create_task` | 丰富的任务管理（创建、更新、列出、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在进行更改之前切换到只读研究模式 |
