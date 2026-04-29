---
name: finishing-a-development-branch
description: 仅在用户明确要求收尾开发分支、决定如何集成当前工作，或明确要求使用 finishing-a-development-branch 时使用
---

# 收尾开发分支

## 概述

通过提供清晰选项并执行用户选择的工作流，来完成开发工作的收尾。

**限制条件：** 即使实现已经完成、测试已经通过，也不能自动使用本技能。只有在用户明确要求收尾、合并、开 PR、保留分支或丢弃工作时，才能启用。

**核心原则：** 验证测试 → 提供选项 → 执行选择 → 清理现场。

**开始时要说明：** “我正在使用 finishing-a-development-branch 技能来完成这项工作。”

## 何时使用

**仅在用户明确要求时使用：**
- 用户明确要求收尾当前开发分支
- 用户明确要求决定接下来是合并、开 PR、保留还是丢弃
- 用户明确要求使用 `finishing-a-development-branch`

**即使符合下列情况，也不能自动使用：**
- 实现已经完成
- 所有测试已经通过
- 你判断现在“应该进入收尾阶段”
- 其他技能流程走到了最后一步

## 流程

### 第 1 步：验证测试

**在提供选项之前，先确认测试通过：**

```bash
# 运行项目测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个失败）。在完成收尾前必须先修复：

[展示失败信息]

在测试通过前，不能继续执行 merge / PR。
```

停止。不要进入第 2 步。

**如果测试通过：** 继续第 2 步。

### 第 2 步：确定基线分支

```bash
# 尝试常见的基线分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或者直接询问：“这个分支是从 main 切出来的吗？”

### 第 3 步：提供选项

严格提供以下 4 个选项：

```
实现已完成。你希望我接下来怎么处理？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保持当前分支不动（我之后再处理）
4. 丢弃这项工作

请选择哪一项？
```

**不要额外解释**，选项应保持简洁。

### 第 4 步：执行选择

#### 选项 1：本地合并

```bash
# 切回基线分支
git checkout <base-branch>

# 拉取最新代码
git pull

# 合并功能分支
git merge <feature-branch>

# 在合并结果上再次验证测试
<test command>

# 如果测试通过
git branch -d <feature-branch>
```

然后：清理 worktree（第 5 步）

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后：清理 worktree（第 5 步）

#### 选项 3：保持现状

报告：`保留分支 <name>。worktree 保持在 <path>。`

**不要清理 worktree。**

#### 选项 4：丢弃

**必须先确认：**
```
这会永久删除以下内容：
- 分支 <name>
- 所有提交：<commit-list>
- 位于 <path> 的 worktree

请输入 'discard' 以确认。
```

等待用户给出完全一致的确认。

如果已确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（第 5 步）

### 第 5 步：清理 worktree

**适用于选项 1、2、4：**

检查当前是否处于 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

如果是：
```bash
git worktree remove <worktree-path>
```

**对于选项 3：** 保留 worktree。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并了损坏代码，或创建了失败的 PR
- **修复方式：** 在提供选项前始终先验证测试

**问题过于开放**
- **问题：** “接下来该怎么做？” → 含糊不清
- **修复方式：** 严格提供这 4 个结构化选项

**自动清理 worktree**
- **问题：** 在仍可能需要时移除了 worktree（选项 2、3）
- **修复方式：** 只在选项 1 和 4 中清理

**丢弃前未确认**
- **问题：** 意外删除工作成果
- **修复方式：** 必须要求输入 `discard` 作为确认

## 风险信号

**绝不要：**
- 在测试失败时继续
- 不验证合并结果测试就直接合并
- 未经确认删除工作
- 在未明确要求的情况下 force-push

**始终要做：**
- 在提供选项前先验证测试
- 严格提供 4 个选项
- 对选项 4 获取明确的输入确认
- 只在选项 1 和 4 中清理 worktree

## 集成关系

**会被以下技能调用：**
- **subagent-driven-development**（第 7 步）- 所有任务完成后
- **executing-plans**（第 5 步）- 所有批次完成后

**常与以下技能配合：**
- **using-git-worktrees** - 清理由该技能创建的 worktree
