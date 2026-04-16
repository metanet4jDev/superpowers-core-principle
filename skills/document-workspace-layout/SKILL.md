---
name: document-workspace-layout
description: Use when design docs, design companion artifacts, implementation plans, planning state files, and task code must follow one task-based directory contract instead of hard-coded paths.
---

# Document Workspace Layout

## Overview

用一个用户维护变量加一个任务名，推导设计文档、详细设计配套文件目录、正式计划文件、`planning-with-files-zh` 三件套，以及 `_tasks/<task-name>` 代码工作目录。调用方先解析路径和当前工作目录，再执行读写或命令。

## When to Use

**Use when:**
- 需要为任务创建或更新设计文档、详细设计配套文件、计划文档时
- 需要让 `planning-with-files-zh` 稳定找到 `task_plan.md`、`findings.md`、`progress.md` 时
- 需要把文档目录和 `_tasks/<task-name>` 代码目录绑定在一起时
- 发现技能或脚本里出现 `docs/superpowers/...`、`harness-hh/spec/...` 这类硬编码路径时

**Do not use when:**
- 只是一次性的临时笔记，不需要进入任务目录规范时
- 当前工作不依赖 `TASK_NAME` 对应的 `_tasks` 代码目录时

## Core Pattern

### 用户维护输入

- `DOC_WORKSPACE_ROOT`：唯一需要长期维护的变量，必须是绝对路径

### 运行时输入

- `TASK_NAME`：必填
- `PLAN_NAME`：可选，按下方规则解析

### 命名约束

- `TASK_NAME`、`PLAN_NAME` 都是目录名，必须匹配 `[A-Za-z0-9._-]+`
- 不得包含 `/`、`\`、`..`
- 不满足约束时立即停止，调用方不得自行清洗后继续

### 派生规则

```text
DOC_PROJECT_DIR    = dirname(${DOC_WORKSPACE_ROOT})
WORKSPACE_ROOT     = dirname(${DOC_PROJECT_DIR})
TASK_DOC_DIR       = ${DOC_WORKSPACE_ROOT}/${TASK_NAME}
SPECS_DIR          = ${TASK_DOC_DIR}/specs
PLANS_DIR          = ${TASK_DOC_DIR}/plans
PLAN_WORKSPACE_DIR = ${PLANS_DIR}/${PLAN_NAME}
DESIGN_DOC_PATH    = ${SPECS_DIR}/design-codex.md
CORE_COGNITION_PATH= ${SPECS_DIR}/core-cognition.md
DESIGN_ARTIFACTS_DIR= ${SPECS_DIR}/artifacts
PLAN_PATH          = ${PLAN_WORKSPACE_DIR}/implementation-plan.md
PLAN_PART_GLOB     = ${PLAN_WORKSPACE_DIR}/implementation-plan-part-*.md
TASK_PLAN_PATH     = ${PLAN_WORKSPACE_DIR}/task_plan.md
FINDINGS_PATH      = ${PLAN_WORKSPACE_DIR}/findings.md
PROGRESS_PATH      = ${PLAN_WORKSPACE_DIR}/progress.md
PLAN_CHANGELOG     = ${PLANS_DIR}/plan-changelog.md
TASKS_ROOT         = ${WORKSPACE_ROOT}/_tasks
TASK_WORKSPACE_DIR = ${TASKS_ROOT}/${TASK_NAME}
```

### `PLAN_NAME` 解析顺序

必须按下面顺序执行，命中即停止：

1. 若调用方显式提供 `PLAN_NAME`，直接使用。
2. 否则，若当前上下文已经落在 `${PLANS_DIR}/<plan-name>/` 下，复用该 `<plan-name>`。
3. 否则，若 `${PLANS_DIR}` 不存在，或存在但没有任何计划子目录，使用 `main`，并允许首次创建 `${PLANS_DIR}/main/`。
4. 否则，若 `${PLANS_DIR}` 下恰好只有一个计划子目录，复用该目录名。
5. 否则立即停止，并要求用户显式指定 `PLAN_NAME`。

### 强制规则

1. 所有调用方在读写文档、生成命令、输出路径前，必须先解析这组路径。
2. `DOC_WORKSPACE_ROOT` 必须是绝对路径，且 basename 必须为 `spec`。不满足时立即停止。
3. `DESIGN_DOC_PATH` 是任务级设计文档唯一合法路径；`CORE_COGNITION_PATH` 是独立核心认知文档唯一合法路径；`DESIGN_ARTIFACTS_DIR` 是详细设计配套文件的默认目录；`PLAN_PATH` 是计划工作区内唯一合法的正式计划入口文件。
4. 当计划需要拆分时，只允许在 `PLAN_WORKSPACE_DIR` 下新增 `implementation-plan-part-*.md` 作为顺序分片文件；`PLAN_PATH` 仍必须保留，并作为唯一入口、唯一 handoff 路径和唯一计划索引文件。
5. `planning-with-files-zh` 三件套必须与 `PLAN_PATH` 同目录存放。
6. `PLAN_CHANGELOG` 是任务级文件，唯一合法路径为 `${PLANS_DIR}/plan-changelog.md`；不得再写任何全局 `.../plans/plan-changelog.md`。
7. 调用 `planning-with-files-zh` 或执行其模板、hooks、脚本前，必须先切换当前工作目录到 `PLAN_WORKSPACE_DIR`。
8. 写入设计文档或详细设计配套文件时，优先切换当前工作目录到 `SPECS_DIR`，并通过相对路径引用 `artifacts/` 下的配套文件。
9. 调用代码检索、构建、测试、实现相关命令前，必须切换当前工作目录到 `TASK_WORKSPACE_DIR`；设计文档、计划文档和代码目录的 cwd 不得混用。
10. `TASK_DOC_DIR`、`SPECS_DIR`、`DESIGN_ARTIFACTS_DIR`、`PLANS_DIR` 允许按需创建。
11. `PLAN_WORKSPACE_DIR` 仅在 `PLAN_NAME` 解析步骤 1 或 3 命中时允许首次创建；其它情况下若目录不存在，立即停止并向用户确认。
12. 初始化三件套的通用脚本不得静默创建缺失的 `PLAN_WORKSPACE_DIR`；只有调用方已明确判定允许首次创建时，才能以显式开关创建。
13. `TASK_WORKSPACE_DIR` 必须预先存在；不得自动创建、不得猜测替代目录。
14. 不要在调用方技能中写死 `docs/superpowers/...`、`harness-hh/spec/...` 或其它项目路径。

## Quick Reference

| 项目 | 规则 |
| --- | --- |
| 最少配置 | `DOC_WORKSPACE_ROOT` |
| 最少运行时信息 | `TASK_NAME` |
| 计划名规则 | 显式输入 > 当前上下文 > 空计划目录默认 `main` > 唯一子目录 > 否则停止 |
| 设计文档 | `${SPECS_DIR}/design-codex.md` |
| 核心认知文档 | `${SPECS_DIR}/core-cognition.md` |
| 详细设计配套文件 | `${SPECS_DIR}/artifacts/` |
| 正式计划文件 | 入口：`${PLAN_WORKSPACE_DIR}/implementation-plan.md`；分片：`${PLAN_WORKSPACE_DIR}/implementation-plan-part-*.md` |
| 三件套位置 | `${PLAN_WORKSPACE_DIR}/` |
| 计划变更日志 | `${PLANS_DIR}/plan-changelog.md` |
| 设计文档 cwd | `${SPECS_DIR}` |
| 计划/tracking cwd | `${PLAN_WORKSPACE_DIR}` |
| 代码类 cwd | `${TASK_WORKSPACE_DIR}` |

## Implementation

```bash
: "${DOC_WORKSPACE_ROOT:?DOC_WORKSPACE_ROOT is required}"
: "${TASK_NAME:?TASK_NAME is required}"

case "$DOC_WORKSPACE_ROOT" in
  /*) ;;
  *) echo "DOC_WORKSPACE_ROOT must be absolute" >&2; exit 1 ;;
esac
[ "$(basename "$DOC_WORKSPACE_ROOT")" = "spec" ] || {
  echo "DOC_WORKSPACE_ROOT must end with /spec" >&2
  exit 1
}

case "$TASK_NAME" in
  *[!/A-Za-z0-9._-]*|*/*|*\\*|*..*) echo "invalid TASK_NAME" >&2; exit 1 ;;
esac
if [ -n "${PLAN_NAME:-}" ]; then
  case "$PLAN_NAME" in
    *[!/A-Za-z0-9._-]*|*/*|*\\*|*..*) echo "invalid PLAN_NAME" >&2; exit 1 ;;
  esac
fi

DOC_PROJECT_DIR="$(dirname "$DOC_WORKSPACE_ROOT")"
WORKSPACE_ROOT="$(dirname "$DOC_PROJECT_DIR")"
TASK_DOC_DIR="$DOC_WORKSPACE_ROOT/$TASK_NAME"
SPECS_DIR="$TASK_DOC_DIR/specs"
PLANS_DIR="$TASK_DOC_DIR/plans"
DESIGN_DOC_PATH="$SPECS_DIR/design-codex.md"
CORE_COGNITION_PATH="$SPECS_DIR/core-cognition.md"
DESIGN_ARTIFACTS_DIR="$SPECS_DIR/artifacts"
TASKS_ROOT="$WORKSPACE_ROOT/_tasks"
TASK_WORKSPACE_DIR="$TASKS_ROOT/$TASK_NAME"
[ -d "$TASK_WORKSPACE_DIR" ] || {
  echo "TASK_WORKSPACE_DIR does not exist" >&2
  exit 1
}

# PLAN_NAME resolution must follow the five-step contract above.
# Do not replace it with PLAN_NAME=${PLAN_NAME:-main}.

PLAN_WORKSPACE_DIR="$PLANS_DIR/$PLAN_NAME"
PLAN_PATH="$PLAN_WORKSPACE_DIR/implementation-plan.md"
PLAN_PART_GLOB="$PLAN_WORKSPACE_DIR/implementation-plan-part-*.md"
TASK_PLAN_PATH="$PLAN_WORKSPACE_DIR/task_plan.md"
FINDINGS_PATH="$PLAN_WORKSPACE_DIR/findings.md"
PROGRESS_PATH="$PLAN_WORKSPACE_DIR/progress.md"
PLAN_CHANGELOG="$PLANS_DIR/plan-changelog.md"
```

设计文档和配套文件相关操作先 `cd "$SPECS_DIR"`；计划、plan review、`planning-with-files-zh` 三件套相关操作先 `cd "$PLAN_WORKSPACE_DIR"`；代码相关操作先 `cd "$TASK_WORKSPACE_DIR"`。不要混用 cwd。

## Common Mistakes

- 把 `PLAN_NAME`、日期、所有派生路径都当成用户要维护的变量。应只让用户维护 `DOC_WORKSPACE_ROOT`，其它都自动推导。
- 把 `task_plan.md`、`findings.md`、`progress.md` 放在任务根目录。它们必须和 `implementation-plan.md` 同目录。
- 把多文件计划做成多个平级入口文件。只能保留一个 `implementation-plan.md` 作为入口；拆分内容只能放进 `implementation-plan-part-*.md`。
- 看到多个计划目录时按名字猜一个继续写。无唯一上下文时必须停止，不能默认回落到 `main`。
- 在设计文档技能里分析代码，却不切到 `${TASK_WORKSPACE_DIR}`。这样文档和实际研发目录会漂移。
- 把类图、时序图、存储说明、API 契约等详细设计配套文件散落在多个临时目录。默认放在 `${DESIGN_ARTIFACTS_DIR}`，再由主设计文档相对引用。
- 只改技能正文，不改 hooks、脚本和 cwd。对 `planning-with-files-zh` 这类技能，这会直接导致三件套写到错误目录。

## Real-World Impact

这条规则的目标不是统一文件名，而是统一“先解路径，再切对 cwd，然后工作”的契约。调用方技能只需要知道任务名和文档根目录，就能把设计、计划、进度记录和 `_tasks/${TASK_NAME}` 代码目录稳定绑在一起。
