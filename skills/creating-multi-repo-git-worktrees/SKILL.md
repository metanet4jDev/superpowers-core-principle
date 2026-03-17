---
name: creating-multi-repo-git-worktrees
description: Use when handling cross-repository tasks that require one combined worktree layout, per-repo hotfix branches, compile-only verification, independent commits, and a final per-repo delivery report.
---

# 创建多仓库 Git Worktree

## 概览

该技能用于处理跨仓库任务：在统一任务目录下创建组合 worktree，并按仓库独立分支、独立编译、独立提交、统一汇总。

**核心原则：** 一个任务目录协同，执行与验证按仓库隔离。

## 强制规则

1. 组合目录：必须在 `<workspace_root>/_tasks/<task-name>/` 下创建 worktree 组合区。
2. 相对结构：必须保持仓库在 `workspace_root` 下的原相对路径。
3. 分支隔离：每个仓库必须独立分支，格式 `hotfix/<task-name>-<suffix>`。
4. AGENTS.md：默认从当前工作目录 `$PWD/AGENTS.md` 复制到任务根目录；若不存在，必须先询问伙伴获取来源路径。
5. 验证方式：只允许 `compile` 或 `package -DskipTests`，禁止 `mvn test`。
6. 提交方式：每个仓库必须独立提交，且提交信息必须遵循 Conventional Commits 1.0.0。
7. 报告输出：最终必须按仓库输出“受影响仓库、编译命令、提交 ID”。

## Conventional Commits 核心规范（作为本技能约束）

来源：<https://www.conventionalcommits.org/zh-hans/v1.0.0/>

提交信息结构：

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

在本技能中，以下条款必须遵守：

1. 每个提交必须有 `type` 前缀，后接冒号和空格。
2. 新功能必须使用 `feat`。
3. Bug 修复必须使用 `fix`。
4. 可选 `scope` 放在 `type` 后圆括号中，如 `fix(parser): ...`。
5. `description` 必须紧跟在 `: ` 后，且为简短变更摘要。
6. 若有正文，正文必须与标题空一行。
7. 若有脚注，脚注与正文空一行，并使用 trailer 风格（如 `Refs: #123`）。
8. 破坏性变更必须标记：要么在头部使用 `!`，要么脚注含 `BREAKING CHANGE: ...`。
9. `BREAKING CHANGE` 必须大写。

推荐类型（除 `feat`/`fix` 外）：`docs`、`refactor`、`perf`、`test`、`build`、`ci`、`chore`。

## 输入约定

开始执行前需要明确：

- `workspace_root`：默认当前工作目录（通常 `/home/haodev/worksapces`）
- `task_name`：任务名
- `repos[]`：受影响仓库相对路径列表
- `parent_repo`：父仓库路径（用于提交子仓库指针）

## 仓库路径自动提取与确认

先从当前对话上下文自动提取仓库路径，再与伙伴确认后执行。

确认模板：

```text
已从上下文提取到以下仓库目录，请确认：
1) <repo-path-1>
2) <repo-path-2>
...
父仓库：<parent-repo>

若需补充，请按以下格式回复：
1) <repo-path>
2) <repo-path>
```

如果伙伴补充仓库，必须按该格式追加并再次确认。

## 执行步骤（严格顺序）

1. 校验仓库有效性：逐个确认路径存在且为 Git 仓库。
2. 创建任务根目录：`<workspace_root>/_tasks/<task-name>/`。
3. 创建仓库 worktree：每个仓库写入任务根目录中对应相对路径。
4. 复制 `AGENTS.md`：优先从 `$PWD/AGENTS.md` 复制到任务根目录。
5. 按仓库编译验证：仅 `compile` 或 `package -DskipTests`。
6. 按仓库独立提交：提交信息使用 Conventional Commits。
7. 父仓库提交：提交子仓库指针变更。
8. 输出报告：按仓库列出编译命令与提交 ID。

## 分支后缀映射建议

- 路径或仓库名含 `access`：`access`
- 路径或仓库名含 `rp`：`rp`
- 其余平台/基础仓库：`plat`

若用户显式给出后缀，以用户要求为准。

## 命令模板

### 初始化任务目录

```bash
workspace_root="/home/haodev/worksapces"
task_name="前置机服务医保上传接口假死"
task_root="$workspace_root/_tasks/$task_name"
mkdir -p "$task_root"
```

### 创建单仓库 worktree（保持相对结构）

```bash
repo_rel="server/yt-pha/yt-pha-rp"
suffix="rp"
repo_abs="$workspace_root/$repo_rel"
branch="hotfix/${task_name}-${suffix}"
wt_path="$task_root/$repo_rel"

mkdir -p "$(dirname "$wt_path")"
git -C "$repo_abs" worktree add "$wt_path" -b "$branch"
```

### 复制 AGENTS.md（当前工作目录优先）

```bash
if [ ! -f "$task_root/AGENTS.md" ]; then
  if [ -f "$PWD/AGENTS.md" ]; then
    cp "$PWD/AGENTS.md" "$task_root/AGENTS.md"
  else
    echo "当前工作目录不存在 AGENTS.md，请先向伙伴确认来源路径"
  fi
fi
```

### Maven 编译验证（禁止测试）

```bash
mvn -s /home/haodev/.m2/whyt-server-settings-linux.xml \
  -DskipTests \
  -DskipAssembly=true \
  -Dmaven.assembly.skipAssembly=true \
  compile

# 或
mvn -s /home/haodev/.m2/whyt-server-settings-linux.xml \
  -DskipTests \
  -DskipAssembly=true \
  -Dmaven.assembly.skipAssembly=true \
  package
```

### 提交示例（Conventional Commits）

```bash
git add <task-related-files>
git commit -m "fix(yt-pha-rp): 前置机服务医保上传接口假死 修复接口假死问题"
```

## 示例映射（题面示例）

- `access/107241/hs-access-server-107241/hs-access-plat` → `hotfix/<task-name>-access`
- `server/basic/hs-hdp` → `hotfix/<task-name>-plat`
- `server/yt-pha/yt-pha-rp` → `hotfix/<task-name>-rp`
- `server/yt-plat` → `hotfix/<task-name>-plat`
- 父仓库：`/home/haodev/worksapces`

## 报告模板

```text
任务：<task_name>

受影响仓库：
1) <repo-1>
2) <repo-2>
...

编译命令（按仓库）：
- <repo-1>: <compile-or-package-command>
- <repo-2>: <compile-or-package-command>

提交 ID（按仓库）：
- <repo-1>: <commit-sha>
- <repo-2>: <commit-sha>
- <parent_repo>: <commit-sha>
```
