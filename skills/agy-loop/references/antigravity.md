# AGY Loop Antigravity workflow contract

This reference defines the required handoffs for the main AGY Loop workflow. The selected role, model, and thinking mode are fixed:

| Stage | Antigravity instance | Model | Thinking |
|---|---|---|---|
| Planning | Planner | Gemini 4 Argon | High |
| Plan review | Separate reviewer instance | Claude Opus 5.5 | Medium |
| Implementation | Developer | Gemini 3.8 Flash | High |
| Each code-change review | A fresh reviewer instance for that change | Claude Opus 5.5 | Medium |

Verify the actual selected model and thinking mode in Antigravity at the start of each stage. If either is unavailable or cannot be verified, stop and report it. Do not silently use a different model, provider, mode, or CLI.

## Reconnaissance and agent/skill inventory

Before writing the plan, inspect relevant source, tests, callers, repository guidance, and available verification commands. Inventory every agent and skill exposed in the developer environment, including workspace-local content such as `.agent`. Use every available agent and skill during reconnaissance. Keep an inventory in the plan or review log, with each agent/skill and how it was used; explicitly record any that cannot be invoked. Do not claim a tool was used based only on its presence.

## Required plan contents

The plan must be concrete enough for independent review and implementation without inventing requirements. Include:

- Goal, acceptance criteria, non-goals, confirmed assumptions, and material risks.
- Inventory of available agents/skills and their use or invocation limitations.
- Verification commands, expected results, and any required manual checks.
- An implementation table with one row per changed function or other atomic change:

| Order | File | Function / change unit | Expected output or observable behavior | Test covering this function/change | Acceptance check |
|---|---|---|---|---|---|
| 1 | `src/module.py` | `function_name` | Exact return/output/state change, including relevant error behavior | `tests/test_module.py::test_function_name_case` | Observable assertion |

Name every file and function that will be edited. For a new function, identify it as new. Specify the expected output/observable behavior for each function, not only the feature's general goal. Give an existing or planned test name for every function; identify the command that runs it. If an entry does not edit a function (for example, configuration or documentation), say `N/A` and explain the applicable validation instead. Do not leave a function without a covering test or an explicit explanation of why a test is inapplicable.

Order work as individual reviewable changes. A change should normally touch one planned function and its directly related test; if an indivisible change needs more, name its exact files/functions and justify the grouping. Any scope change requires updating and re-approving the plan before implementation.

## Plan approval

The planner submits the complete plan to a separate, read-only Claude Opus 5.5 medium-thinking instance. The reviewer checks every planned file/function, expected behavior, test mapping, assumptions, and acceptance criteria against the repository. It returns:

- Verdict: `APPROVED`, `REVISE`, or `BLOCKED`.
- Evidence-backed findings with severity, affected plan entry/path, scenario, and required correction.
- Actual coverage and limitations.

`APPROVED` means no unresolved material defect; it is not proof of exhaustive correctness. The planner records the approved plan's exact path and SHA256 in the append-only review log. Any plan edit invalidates approval and requires review of the revised plan. No code change starts until the current plan has an explicit `APPROVED` verdict.

## One-change / one-review gate

The developer implements one planned change, runs its named test, and pauses. A new Claude Opus 5.5 medium-thinking Antigravity instance reviews that change before any next planned change begins. Supply the reviewer with the approved plan entry and hash, the diff since the last approved checkpoint, relevant surrounding code, and actual test output. The review is read-only and returns a verdict, findings, inspected coverage, and limitations.

- `APPROVED`: record the exact reviewed diff/checkpoint and proceed to the next planned change.
- `REVISE`: the developer applies only the necessary correction, reruns affected tests, and requests review from another fresh Opus instance. Repeat until approved or blocked.
- `BLOCKED` or failed review: stop; do not treat it as approval or continue with the next change.

Each developer edit, including a correction requested by review, requires its own new reviewer instance. Never combine several planned changes before review, reuse a reviewer instance for a later code change, or count a review of one snapshot as approval of later edits. After all per-change approvals, run the full proof commands and inspect the complete final diff.
