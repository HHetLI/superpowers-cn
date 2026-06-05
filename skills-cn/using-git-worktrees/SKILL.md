---
name: using-git-worktrees
description: 当开始需要与当前工作区隔离的功能开发时，或在执行实现计划之前使用——通过原生工具或 git worktree 回退方案确保隔离的工作区存在
---

# 使用 Git Worktrees

## 概述

确保工作在隔离的工作区中进行。优先使用你平台原生的 worktree 工具。仅当没有原生工具可用时，才回退到手动 git worktree。

**核心原则：** 先检测已有的隔离环境。然后使用原生工具。最后回退到 git。永远不要与harness对抗。

**开始时声明：** "我正在使用 using-git-worktrees 技能来设置隔离工作区。"

## 步骤 0：检测已有隔离环境

**在创建任何东西之前，先检查你是否已经在隔离的工作区中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**Submodule 防护：** `GIT_DIR != GIT_COMMON` 在 git submodule 中也成立。在得出"已在 worktree 中"的结论之前，先验证你不在 submodule 中：

```bash
# 如果这条命令返回一个路径，说明你在 submodule 中，而不是 worktree —— 按普通仓库处理
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`（且不是 submodule）：** 你已在一个关联的 worktree 中。跳到步骤 3（项目设置）。不要创建另一个 worktree。

报告分支状态：
- 在分支上："已在 <路径> 的隔离工作区中，位于 <名称> 分支上。"
- Detached HEAD："已在 <路径> 的隔离工作区中（分离 HEAD，外部管理）。完成时需要创建分支。"

**如果 `GIT_DIR == GIT_COMMON`（或在 submodule 中）：** 你在一个普通的仓库检出中。

用户是否在你的指令中已经表明了 worktree 偏好？如果没有，在创建 worktree 之前先征得同意：

> "Would you like me to set up an isolated worktree? It protects your current branch from changes."

尊重任何已声明的偏好，不再询问。如果用户拒绝，就在当前目录工作，跳到步骤 3。

## 步骤 1：创建隔离工作区

**你有两种机制。按以下顺序尝试。**

### 1a. 原生 Worktree 工具（首选）

用户已要求一个隔离工作区（步骤 0 同意）。你是否已经有创建 worktree 的方式？它可能是一个名为 `EnterWorktree`、`WorktreeCreate` 的工具，一个 `/worktree` 命令，或一个 `--worktree` 标志。如果有，使用它并跳到步骤 3。

原生工具会自动处理目录位置、分支创建和清理。当你拥有原生工具时使用 `git worktree add`，会创建你的harness无法看到或管理的幽灵状态。

仅当你没有原生 worktree 工具可用时，才继续步骤 1b。

### 1b. Git Worktree 回退

**仅当步骤 1a 不适用时使用** —— 你没有原生 worktree 工具可用。手动使用 git 创建 worktree。

#### 目录选择

按以下优先级顺序。用户的显式偏好始终优先于已观察到的文件系统状态。

1. **检查你的指令中是否有声明的 worktree 目录偏好。** 如果用户已经指定了，直接使用，无需询问。

2. **检查是否存在项目本地的 worktree 目录：**
   ```bash
   ls -d .worktrees 2>/dev/null     # 首选（隐藏目录）
   ls -d worktrees 2>/dev/null      # 备选
   ```
   如果找到了，使用它。如果两者都存在，`.worktrees` 优先。

3. **检查是否存在全局目录：**
   ```bash
   project=$(basename "$(git rev-parse --show-toplevel)")
   ls -d ~/.config/superpowers/worktrees/$project 2>/dev/null
   ```
   如果找到了，使用它（向后兼容旧的全局路径）。

4. **如果没有其他指导可用**，默认使用项目根目录下的 `.worktrees/`。

#### 安全性验证（仅限项目本地目录）

**在创建 worktree 之前必须验证目录已被忽略：**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：** 添加到 .gitignore，提交更改，然后继续。

**为什么至关重要：** 防止意外将 worktree 内容提交到仓库。

全局目录（`~/.config/superpowers/worktrees/`）无需验证。

#### 创建 Worktree

```bash
project=$(basename "$(git rev-parse --show-toplevel)")

# 根据选择的位置确定路径
# 项目本地：path="$LOCATION/$BRANCH_NAME"
# 全局：path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退：** 如果 `git worktree add` 由于权限错误（沙箱拒绝）而失败，告诉用户沙箱阻止了 worktree 创建，你将在当前目录中工作。然后在原地运行设置和基线测试。

## 步骤 3：项目设置

自动检测并运行适当的设置：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 步骤 4：验证清洁基线

运行测试以确保工作区从清洁状态开始：

```bash
# 使用适合项目的命令
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：** 报告失败，询问是继续还是调查。

**如果测试通过：** 报告就绪。

### 报告

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| 已在关联 worktree 中 | 跳过创建（步骤 0） |
| 在 submodule 中 | 按普通仓库处理（步骤 0 防护） |
| 原生 worktree 工具可用 | 使用它（步骤 1a） |
| 没有原生工具 | Git worktree 回退（步骤 1b） |
| `.worktrees/` 存在 | 使用它（验证已忽略） |
| `worktrees/` 存在 | 使用它（验证已忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 检查指令文件，然后默认 `.worktrees/` |
| 全局路径存在 | 使用它（向后兼容） |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| 创建时权限错误 | 沙箱回退，原地工作 |
| 基线测试失败 | 报告失败 + 询问 |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 与harness对抗

- **问题：** 当平台已经提供隔离时使用 `git worktree add`
- **修复：** 步骤 0 检测已有的隔离。步骤 1a 优先使用原生工具。

### 跳过检测

- **问题：** 在已有 worktree 内部创建嵌套 worktree
- **修复：** 在创建任何东西之前始终运行步骤 0

### 跳过忽略验证

- **问题：** Worktree 内容被追踪，污染 git 状态
- **修复：** 在创建项目本地 worktree 之前始终使用 `git check-ignore`

### 假设目录位置

- **问题：** 造成不一致，违反项目约定
- **修复：** 遵循优先级：现有 > 全局遗留 > 指令文件 > 默认

### 在测试失败时继续

- **问题：** 无法区分新的 bug 和已有的问题
- **修复：** 报告失败，获得明确许可后再继续

## 红旗信号 (Red Flags)

**永远不要：**
- 当步骤 0 检测到已有隔离时创建 worktree
- 当你有原生 worktree 工具（例如 `EnterWorktree`）时使用 `git worktree add`。这是头号错误 —— 如果你有它，使用它。
- 跳过步骤 1a 直接跳到步骤 1b 的 git 命令
- 在未验证目录已被忽略的情况下创建 worktree（项目本地）
- 跳过基线测试验证
- 在未询问的情况下继续处理测试失败

**始终：**
- 首先运行步骤 0 检测
- 优先使用原生工具而非 git 回退
- 遵循目录优先级：现有 > 全局遗留 > 指令文件 > 默认
- 对项目本地目录验证已忽略
- 自动检测并运行项目设置
- 验证清洁的测试基线
