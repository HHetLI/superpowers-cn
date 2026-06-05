---
name: finishing-a-development-branch
description: 当实现完成、所有测试通过，需要决定如何集成工作时使用——通过呈现合并、PR 或清理的结构化选项来指导开发工作的收尾
---

# 完成开发分支

## 概述

通过呈现清晰的选项并处理所选工作流来指导开发工作的收尾。

**核心原则：** 验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理。

**开始时声明：** "我正在使用 finishing-a-development-branch 技能来完成此工作。"

## 流程

### 步骤 1：验证测试

**在呈现选项之前，先验证测试通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个）。必须在完成前修复：

[显示失败项]

在测试通过之前无法继续合并/PR。
```

停止。不要继续步骤 2。

**如果测试通过：** 继续步骤 2。

### 步骤 2：检测环境

**在呈现选项之前确定工作区状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这决定了要显示哪个菜单以及清理方式：

| 状态 | 菜单 | 清理 |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准 4 选项 | 无 worktree 需要清理 |
| `GIT_DIR != GIT_COMMON`，命名分支 | 标准 4 选项 | 基于来源（provenance）的清理（见步骤 6） |
| `GIT_DIR != GIT_COMMON`，分离 HEAD（detached HEAD） | 缩减 3 选项（无合并） | 无清理（外部管理） |

### 步骤 3：确定基准分支

```bash
# 尝试常见的基准分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："此分支是从 main 分出的——对吗？"

### 步骤 4：呈现选项

**普通仓库和命名分支 worktree——严格呈现以下 4 个选项：**

```
实现已完成。您想如何进行？

1. 合并回 <基准分支>（本地）
2. 推送并创建 Pull Request
3. 保持分支不变（我稍后处理）
4. 丢弃此工作

请选择？
```

**分离 HEAD——严格呈现以下 3 个选项：**

```
实现已完成。您处于分离 HEAD 状态（外部管理的工作区）。

1. 作为新分支推送并创建 Pull Request
2. 保持现状（我稍后处理）
3. 丢弃此工作

请选择？
```

**不要添加解释**——保持选项简洁。

### 步骤 5：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根目录以确保 CWD 安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并——在移除任何内容之前验证成功
git checkout <base-branch>
git pull
git merge <feature-branch>

# 验证合并结果上的测试
<test command>

# 仅在合并成功后：清理 worktree（步骤 6），然后删除分支
```

然后：清理 worktree（步骤 6），再删除分支：

```bash
git branch -d <feature-branch>
```

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<2-3 点说明变更内容>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

**不要清理 worktree**——用户需要保留它以便根据 PR 反馈迭代。

#### 选项 3：保持现状

报告："保留分支 <名称>。Worktree 保留在 <路径>。"

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <名称>
- 所有提交：<提交列表>
- Worktree 位于 <路径>

请输入 'discard' 确认。
```

等待精确确认。

确认后：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理 worktree（步骤 6），再强制删除分支：
```bash
git branch -D <feature-branch>
```

### 步骤 6：清理工作区

**仅对选项 1 和 4 执行。** 选项 2 和 3 始终保留 worktree。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：** 普通仓库，无需清理 worktree。完成。

**如果 worktree 路径位于 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下：** Superpowers 创建了此 worktree——由我们负责清理。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自愈：清理任何残留注册
```

**否则：** harness拥有此工作区。不要移除它。如果你的平台提供工作区退出工具，请使用它。否则，将工作区保留在原位。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持现状 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏的代码，创建失败的 PR
- **修复：** 始终在提供选项之前验证测试

**开放式问题**
- **问题：** "接下来我应该做什么？" 表述模糊
- **修复：** 严格呈现 4 个结构化选项（分离 HEAD 则为 3 个）

**为选项 2 清理 worktree**
- **问题：** 移除了用户 PR 迭代所需的 worktree
- **修复：** 仅对选项 1 和 4 执行清理

**在移除 worktree 之前删除分支**
- **问题：** `git branch -d` 失败，因为 worktree 仍然引用该分支
- **修复：** 先合并，再移除 worktree，然后删除分支

**在 worktree 内部运行 `git worktree remove`**
- **问题：** 当 CWD 在要移除的 worktree 内部时，命令静默失败
- **修复：** 始终在 `git worktree remove` 之前先 `cd` 到主仓库根目录

**清理宿主创建的 worktree**
- **问题：** 移除宿主创建的 worktree 会导致幽灵状态
- **修复：** 仅清理位于 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下的 worktree

**丢弃时无确认**
- **问题：** 意外删除工作成果
- **修复：** 要求输入 "discard" 确认

## 红旗信号 (Red Flags)

**绝不：**
- 测试失败时继续
- 未验证合并结果上的测试就合并
- 未经确认就删除工作
- 未经明确请求就强制推送
- 在确认合并成功之前移除 worktree
- 清理不是你创建的 worktree（来源检查）
- 在 worktree 内部运行 `git worktree remove`

**始终：**
- 在提供选项之前验证测试
- 在呈现菜单之前检测环境
- 严格呈现 4 个选项（分离 HEAD 则为 3 个）
- 对选项 4 要求输入确认
- 仅对选项 1 和 4 清理 worktree
- 在 worktree 移除前 `cd` 到主仓库根目录
- 移除后运行 `git worktree prune`
