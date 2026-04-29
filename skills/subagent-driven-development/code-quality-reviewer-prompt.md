# 代码质量评审提示词模板

仅在用户明确要求做代码质量评审时使用此模板。不要在实现任务完成后自动派发。

**目的：** 验证实现是否构建得足够好，是否整洁、可测试、可维护

**只有在用户明确要求代码质量评审后，才能派发。** 如果用户同时要求规格符合性评审，只有在规格符合性评审通过后，才能进行代码质量评审。

```
Task tool（superpowers:code-reviewer）：
  使用模板：requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [来自实现代理的报告]
  PLAN_OR_REQUIREMENTS: [plan-file] 中的任务 N
  BASE_SHA: [任务开始前的提交]
  HEAD_SHA: [当前提交]
  DESCRIPTION: [任务摘要]
```

**除了标准代码质量问题外，reviewer 还应检查：**
- 每个文件是否都只有一个清晰职责，并暴露定义明确的接口？
- 各个单元是否被合理拆分，以便独立理解和测试？
- 实现是否遵循了计划里规定的文件结构？
- 这次实现是否新增了已经偏大的新文件，或让现有文件明显继续膨胀？（不要因为既有文件本来就大而报问题，重点关注“本次改动新增了什么负担”）

**代码评审代理应返回：** Strengths、Issues（Critical / Important / Minor）、Assessment
