# Codex 工具映射

技能使用 Claude Code 的工具名称。当你在技能中遇到这些名称时，使用你平台的等价物：

| 技能引用 | Codex 等价物 |
|-----------------|------------------|
| `Task` 工具（分派subagent） | `spawn_agent`（参见[subagent分派需要多agent支持](#subagent分派需要多agent支持)） |
| 多个 `Task` 调用（并行） | 多个 `spawn_agent` 调用 |
| Task 返回结果 | `wait_agent` |
| Task 自动完成 | `close_agent` 释放槽位 |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` 工具（调用技能） | 技能原生加载 —— 直接遵循指令 |
| `Read`、`Write`、`Edit`（文件） | 使用你的原生文件工具 |
| `Bash`（运行命令） | 使用你的原生 Shell 工具 |

## subagent分派需要多agent支持

添加到你的 Codex 配置（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

这为 `dispatching-parallel-agents` 和 `subagent-driven-development` 等技能启用了 `spawn_agent`、`wait_agent` 和 `close_agent`。

遗留说明：`rust-v0.115.0` 之前的 Codex 构建将已生成agent的等待暴露为 `wait`。当前 Codex 使用 `wait_agent` 来处理已生成的agent。`wait` 名称现在属于代码模式的 `exec/wait`，它通过 `cell_id` 恢复已让出的执行单元；它不是已生成agent的结果工具。

## 环境检测

创建 worktree 或完成分支的技能应在继续之前使用只读的 git 命令检测其环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已在一个关联的 worktree 中（跳过创建）
- `BRANCH` 为空 → detached HEAD（无法从沙箱分支/推送/PR）

参见 `using-git-worktrees` 步骤 0 和 `finishing-a-development-branch` 步骤 1 了解每个技能如何使用这些信号。

## Codex App 完成

当沙箱阻止分支/推送操作（在外部管理的 worktree 中处于 detached HEAD），agent会提交所有工作并告知用户使用 App 的原生控件：

- **"Create branch"** — 命名分支，然后通过 App UI 提交/推送/PR
- **"Hand off to local"** — 将工作转移到用户的本地检出

agent仍然可以运行测试、暂存文件，并输出建议的分支名称、提交消息和 PR 描述供用户复制。
