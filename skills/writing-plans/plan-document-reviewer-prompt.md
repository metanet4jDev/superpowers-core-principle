# Plan Document Reviewer Prompt Template

Use this template only when the user explicitly asks to review an implementation plan. Do not dispatch it automatically after writing or updating a plan.

**Purpose:** Verify the plan is complete, executable, and strictly aligned with the design/core-cognition source documents.

**Dispatch after:** Each plan chunk is written, or after the full plan is written if it is small.

```
Task tool (general-purpose):
  description: "Review implementation plan"
  prompt: |
    You are an implementation plan reviewer. Review the provided plan artifacts only. Do not rely on conversation history.

    **Plan entry:** [PLAN_PATH]
    **Current chunk or part:** [PLAN_CHUNK_OR_PART_PATH]
    **Design document:** [DESIGN_DOC_PATH]
    **Core cognition document:** [CORE_COGNITION_PATH]
    **Companion artifacts:** [LIST_PATHS]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Source alignment | No additions, deletions, or semantic changes relative to design/core cognition |
    | Source references | Every task/step cites source sections such as `Design §x.y` or `Core Cognition §a.b` |
    | Entry contract | `implementation-plan.md` is the single entry; split parts are ordered and linked |
    | Task sequencing | Mandatory sequencing gate exists; each task ends by marking itself complete |
    | Task decomposition | Each task is small, serial, independently meaningful, and actionable |
    | TDD | Tests are written and run before implementation where behavior changes |
    | Buildability | Commands, expected outputs, files, code snippets, and verification steps are concrete |
    | Comments | Implementation steps explicitly require code comments aligned to source docs |
    | Placeholders | No TODO/TBD/fill-in/similar-to/hand-wavy steps |
    | File limits | Each file <=1000 lines; each `## Chunk N` <=500 lines |

    ## Calibration

    Flag only issues that would cause an implementer to build the wrong thing, get stuck, skip verification, or drift from the design.
    Minor wording and style preferences are advisory only.

    ## Required Output

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Blocker|Major|Minor] [Task/Step/Section]: [specific issue]
      - Evidence: <specific plan text or missing item>
      - Violated rule: <which plan rule is violated>
      - Fix direction: <how to fix>
      - Verification: <how to confirm the fix worked>

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, blocking issues, advisory recommendations.
