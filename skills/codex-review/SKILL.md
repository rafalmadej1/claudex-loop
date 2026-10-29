---
name: codex-review
description: "Legacy Claude Code/Codex CLI compatibility command for reviewing an existing plan; the main AGY Loop workflow uses Antigravity."
---

# Codex Review

Legacy compatibility entry point: explicitly use Codex as the plan reviewer in the Claude Code/Codex CLI workflow. This command is separate from the main Antigravity workflow and does not satisfy its plan-approval or per-change review gates.

In Claude Code, use the shared CLI runner with `review --host claude` to ask Codex to review the supplied plan. Preserve the plan path, log path, round cap, and explicit model/effort arguments. The invocation authorizes reviewing, not implementation. Do not restart an interview when the supplied plan is already adequate.

In Codex, explain that this explicit command would use the same provider for planning and review. Use an ordinary Codex review only if the user requested same-provider review; do not claim cross-provider approval.

Read the shared [runtime reference](../agy-loop/references/runtime.md). Resolve the runner from the installed sibling skill, not the target repository. A valid completed result and `APPROVED` verdict are both required; failed, empty, blocked, or malformed results are not approval. The host logs findings and dispositions and stops at the round cap.

If implementation follows this legacy workflow, use its corresponding build/inspection process. This does not replace the required Antigravity workflow when invoked through `agy-loop`.
