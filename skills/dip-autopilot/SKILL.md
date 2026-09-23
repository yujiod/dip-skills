---
name: dip:autopilot
description: Full autonomous execution from product idea to working, verified code with subagent-gated pipeline
argument-hint: "[--quick|--deep] [--skip-qa] [--skip-review] <product idea or task description>"
pipeline: [dip:autopilot]
handoff-policy: approval-required
level: 4
---

# Autopilot (`dip:autopilot`)

<Purpose>
`dip:autopilot` takes a brief product idea or high-level task and autonomously orchestrates the complete engineering lifecycle: requirements expansion, consensus planning, subagent execution, QA cycling, and multi-perspective validation. It produces working, verified code without cutting corners or allowing single-agent self-agreement.
</Purpose>

<Use_When>
- User wants end-to-end autonomous execution from an idea to working code
- User says "autopilot", "auto pilot", "build me", "create me", "full auto", or "handle it all"
- Task spans multiple lifecycle stages: specification, planning, coding, QA, and validation
</Use_When>

<Do_Not_Use_When>
- User wants conversational exploration or brainstorming only -- respond conversationally
- User requests a minor single-file fix -- use `dip:execute` directly
- User wants only to review an existing plan -- use `dip:deep-plan`
</Do_Not_Use_When>

<Execution_Policy>
1. **Mandatory Subagent Separation (Strictly No Self-Agreement Across All Phases)**:
   - Self-agreement, self-implementation, or self-review within the lead agent context is **strictly prohibited**.
   - Every phase leverages dedicated, isolated subagents dispatched via `invoke_subagent`, except for user-facing Socratic expansion in Phase 0.
   - **Expansion (Phase 0)**: Lead agent executes `dip:deep-interview` runbook directly using `ask_question` (interactive user dialogue requires lead context; it is NOT a subagent TypeName).
   - **Planning (Phase 1)**: Planner (Lead) + Architect subagent + Critic subagent (`dip:deep-plan`).
   - **Execution (Phase 2)**: Dispatched Code Executor subagents (`dip:execute`).
   - **QA (Phase 3)**: Dispatched QA Engineer subagent (`dip:verify`).
   - **Validation (Phase 4)**: 3 parallel reviewer subagents: Architect, Security, Code (`dip:review`).
2. **Phase Completion Gate**:
   - Each phase must satisfy its verification gate before the next phase begins.
3. **Hybrid Smart Progression**:
   - If an approved specification (`.dip/specs/deep-interview-*.md`) or consensus plan (`.dip/plans/plan-*.md`) already exists in workspace, skip Phase 0 and/or Phase 1 and jump directly to Phase 2 (Execution).
   - If starting from an ambiguous prompt, initiate `dip:deep-interview` before architectural planning.
4. **Escalation & Stop Conditions**:
   - Stop and report when the same QA error persists across 3 consecutive cycles.
   - Stop and report when validation fails across 3 re-validation rounds.
   - Stop immediately if the user requests cancellation.
5. **State Discipline & Cleanup**:
   - Track session progress in `.dip/state/autopilot-state.json`.
   - On successful validation, remove temporary state files and present the final deliverables report to the user.
</Execution_Policy>

---

## The 5-Phase End-to-End Lifecycle

```
[User Idea / Task]
        │
        v
   [Phase 0: Expansion] ─────────► dip:deep-interview (Socratic Q&A, Ambiguity <= 20%)
        │                          (Skipped if valid .dip/specs/ exists)
        v
   [Phase 1: Planning] ──────────► dip:deep-plan (RALPLAN-DR Consensus: Architect & Critic)
        │                          (Skipped if valid .dip/plans/ exists)
        v
   [Explicit User Approval Gate]
        │
        v
   [Phase 2: Execution] ─────────► dip:execute (Dispatched Executor Subagents)
        │
        v
   [Phase 3: QA Cycling] ────────► dip:verify (QA Subagent: Build -> Lint -> Test -> Fix)
        │                          (Max 5 cycles, halt on 3 identical errors)
        v
   [Phase 4: Validation] ────────► dip:review (Parallel: Architect + Security + Code Subagents)
        │                          (Max 3 rounds, unanimous approval required)
        v
   [Phase 5: Cleanup & Delivery] ─► Clean .dip/state/, summarize deliverables to user
```

---

## Detailed Phases

### Phase 0: Expansion (`dip:deep-interview`)
- **Runbook**: Read instructions using `view_file` on [skills/dip-deep-interview/SKILL.md](../dip-deep-interview/SKILL.md) (or use the exact path from system prompt `Available skills`).
- **Execution Mode**: Direct execution by Lead Agent using `ask_question` (do NOT invoke as a subagent TypeName).
- **Input Check**: Inspect `.dip/specs/` for an existing specification.
- If existing spec is found and valid: reuse immediately and skip to Phase 1.
- If vague or new:
  - Prompt user: "Request received. Starting deep interview to crystallize requirements."
  - Execute `dip:deep-interview` Socratic loop via `ask_question` until Ambiguity $\le 20\%$ (or configured threshold).
  - Yields `.dip/specs/deep-interview-{slug}.md`.

### Phase 1: Planning (`dip:deep-plan`)
- **Runbook**: Read instructions using `view_file` on [skills/dip-deep-plan/SKILL.md](../dip-deep-plan/SKILL.md) (or use the exact path from system prompt `Available skills`).
- **Input Check**: Inspect `.dip/plans/` for an existing plan matching the spec.
- If existing approved plan exists: skip directly to Phase 2.
- Otherwise:
  - Formulate draft plan with RALPLAN-DR framework.
  - Dispatch Architect and Critic subagents via `invoke_subagent` (`Model: "pro"`).
  - Iterate until unanimous consensus is recorded.
  - Yields `.dip/plans/plan-{slug}.md` (`Status: PENDING APPROVAL`).
  - **Approval Gate**: Prompt user for explicit approval to begin execution.

### Phase 2: Execution (`dip:execute`)
- **Runbook**: Read instructions using `view_file` on [skills/dip-execute/SKILL.md](../dip-execute/SKILL.md) (or use the exact path from system prompt `Available skills`).
- Read approved `.dip/plans/plan-{slug}.md`.
- Break plan into atomic work milestones.
- Dispatch implementation tasks to isolated executor subagents via `invoke_subagent` (`TypeName: "self"`, `Role: "Code Executor"`).
- Run independent components in parallel if `--parallel` is active.

### Phase 3: QA Cycling (`dip:verify`)
- **Runbook**: Read instructions using `view_file` on [skills/dip-verify/SKILL.md](../dip-verify/SKILL.md) (or use the exact path from system prompt `Available skills`).
- Dispatch QA Engineer subagent via `invoke_subagent` (`TypeName: "self"`, `Role: "QA Engineer"`).
- Run project build, lint, and test suites.
- If failures occur: diagnose and apply targeted fixes (up to 5 cycles).
- **Guardrail**: If the identical error signature occurs 3 times, abort and escalate to user.

### Phase 4: Validation (`dip:review`)
- **Runbook**: Read instructions using `view_file` on [skills/dip-review/SKILL.md](../dip-review/SKILL.md) (or use the exact path from system prompt `Available skills`).
- Dispatch 3 independent reviewer subagents in parallel via `invoke_subagent` (`Model: "pro"`):
  1. **Architect Reviewer**: Verifies plan compliance and interface boundaries.
  2. **Security Reviewer**: Verifies OWASP, credential safety, and data sanitization.
  3. **Code Reviewer**: Verifies maintainability, edge cases, and removes AI slop.
- Require unanimous `APPROVE`.
- If issues are flagged: dispatch executor subagent to fix, re-verify with `dip:verify`, and re-review (up to 3 rounds).

### Phase 5: Cleanup & Delivery
- When all 3 reviewers approve:
  - Record final review at `.dip/reviews/review-{slug}.md`.
  - Remove transient state in `.dip/state/autopilot-state.json`.
  - Present final summary to user:
    - Confirmed requirements and specs
    - Modified/created files
    - Test execution evidence
    - Multi-perspective review sign-offs
