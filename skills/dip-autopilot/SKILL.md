---
name: dip:autopilot
description: Full autonomous execution from product idea to working, verified code with subagent-gated pipeline
argument-hint: "[--quick|--deep] [--skip-qa] [--skip-review] <product idea or task description>"
pipeline: [dip:deep-interview, dip:deep-plan, dip:execute, dip:verify, dip:review]
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
1. **Lifecycle Architecture & Subagent Separation**:
   - **Phase 0: Socratic Expansion (`dip:deep-interview`) - Lead Agent Skill Activation**:
     - The interactive UI modal tool (`ask_question`) requires the interactive main chat session and **CANNOT be executed by background subagents**.
     - Therefore, the **Lead agent MUST directly execute the `dip:deep-interview` skill workflow**.
     - **Tool Sequence Hard Gate (Zero Shortcut Rule)**: Calling `ask_question` directly without prior codebase exploration and ambiguity calculation is **STRICTLY PROHIBITED**. On Turn 1, the Lead agent MUST load [skills/dip-deep-interview/SKILL.md](../dip-deep-interview/SKILL.md) via `view_file`, explore codebase context, announce the interview start with initial 4-dimension scores, and only then proceed to Socratic inquiry via `ask_question`.
     - Skipping `dip:deep-interview` is strictly prohibited.
   - **Phases 1 - 4: Mandatory Subagent Separation (Strictly No Self-Agreement)**:
     - Once the specification is established, self-agreement, self-implementation, or self-review within the lead agent context is **strictly prohibited**.
     - Every subsequent phase leverages dedicated, isolated subagents dispatched via `invoke_subagent`:
       - **Expansion (Phase 0)**: Lead + [`dip-explore`](../../agents/dip-explore.md) + [`dip-analyst`](../../agents/dip-analyst.md) + [`dip-critic`](../../agents/dip-critic.md).
       - **Planning (Phase 1)**: Planner (Lead / [`agents/dip-planner.md`](../../agents/dip-planner.md)) + Architect subagent ([`agents/dip-architect.md`](../../agents/dip-architect.md)) + Critic subagent ([`agents/dip-critic.md`](../../agents/dip-critic.md)) (`dip:deep-plan`).
       - **Execution (Phase 2)**: Dispatched Code Executor subagents ([`agents/dip-executor.md`](../../agents/dip-executor.md)) + [`dip-code-simplifier`](../../agents/dip-code-simplifier.md) + [`dip-git-master`](../../agents/dip-git-master.md) (`dip:execute`).
       - **QA (Phase 3)**: Dispatched QA Engineer subagent ([`agents/dip-qa-tester.md`](../../agents/dip-qa-tester.md)) + [`dip-debugger`](../../agents/dip-debugger.md) + [`dip-verifier`](../../agents/dip-verifier.md) (`dip:verify`).
       - **Validation (Phase 4)**: 3 parallel reviewer subagents: Architect ([`agents/dip-architect.md`](../../agents/dip-architect.md)), Security ([`agents/dip-security-reviewer.md`](../../agents/dip-security-reviewer.md)), Code ([`agents/dip-code-reviewer.md`](../../agents/dip-code-reviewer.md)) (`dip:review`).
2. **Skill Resolution Protocol (Discovery Order)**:
   - When loading runbooks for any phase via `view_file`, resolve paths in the following priority order:
     1. Exact path listed in system prompt `Available skills` (highest priority)
     2. Global user installation: `~/.agents/skills/dip-<name>/SKILL.md`
     3. Project installation: `.agents/skills/dip-<name>/SKILL.md`
     4. Repository workspace: `skills/dip-<name>/SKILL.md` (or relative path `../dip-<name>/SKILL.md`)
3. **Phase Completion Gate**:
   - Each phase must satisfy its verification gate before the next phase begins.
4. **Hybrid Smart Progression & Strict Gating**:
   - If an approved specification (`.dip/specs/deep-interview-*.md`) already exists in workspace, reuse it and jump to Phase 1.
   - If an approved consensus plan (`.dip/plans/plan-*.md`) already exists in workspace, skip Phase 0 & 1 and jump directly to Phase 2 (Execution).
   - **NO SELF-EXEMPTION (Zero-Skip Rule)**: The Lead agent is STRICTLY PROHIBITED from skipping `dip:deep-interview` by claiming the user's prompt is "already clear", "detailed", or "unambiguous". Every new request contains implicit assumptions. The Lead agent MUST execute `dip:deep-interview` to measure ambiguity and generate `.dip/specs/deep-interview-{slug}.md`.
5. **Escalation & Stop Conditions**:
   - Stop and report when the same QA error persists across 3 consecutive cycles.
   - Stop and report when validation fails across 3 re-validation rounds.
   - Stop immediately if the user requests cancellation.
6. **State Discipline & Cleanup**:
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

### Phase 0: Expansion (`dip:deep-interview`) - MANDATORY LEAD ACTIVATION
- **Session Constraint**: Interactive UI modals (`ask_question`) can ONLY be presented by the interactive main chat session; background subagents cannot interact with the user. Therefore, the Lead agent directly executes the `dip:deep-interview` skill.
- **Strict Execution Sequence (Zero-Shortcut Rule)**:
  Calling `ask_question` blindly without following the protocol is a **CRITICAL PROTOCOL VIOLATION**. The Lead agent MUST strictly follow this exact 4-step sequence:
  1. **Step 1 - Skill Ingestion (Turn 1)**: Immediately read `dip:deep-interview` runbook using `view_file` (resolve path: Available skills -> `~/.agents/skills/dip-deep-interview/SKILL.md` -> `.agents/skills/dip-deep-interview/SKILL.md` -> `skills/dip-deep-interview/SKILL.md`).
  2. **Step 2 - Context & Brownfield Exploration**: Inspect workspace code, tests, and any prior `.dip/specs/` or `.dip/plans/`. Never question the user regarding existing code without checking it first.
  3. **Step 3 - Interview Announcement & Ambiguity Scoring**: Output visible text in the chat declaring:
     - Header: `### [Phase 0: dip:deep-interview] 要件展開開始`
     - Analysis: Project type (Greenfield vs Brownfield) & codebase context findings.
     - Ambiguity Score: Initial scoring across the 4 weighted dimensions ($W_{goal}=0.30$, $W_{constraints}=0.25$, $W_{criteria}=0.25$, $W_{context}=0.20$) and target threshold ($\le 20\%$).
     - Scope Topology: Candidate components identified (1 to 6 components).
  4. **Step 4 - Socratic Inquiry via `ask_question`**: Present Round 0 (Topology confirmation) or the weakest dimension's targeted question via `ask_question`.
- **Specification Persistence**: Repeat the inquiry loop until $\text{Ambiguity} \le 20\%$. Persist the finalized specification artifact to `.dip/specs/deep-interview-{slug}.md`.
- **Hard Gate Rule**:
  - ONLY skip Phase 0 if a valid, approved specification file (`.dip/specs/deep-interview-*.md`) already exists in workspace.
  - Moving to Phase 1 (Planning) or Phase 2 (Execution) without completing `dip:deep-interview` and generating `.dip/specs/deep-interview-{slug}.md` is a **CRITICAL PROTOCOL VIOLATION**.
  - **Zero Self-Exemption**: Never judge the user prompt as "already clear" or "sufficiently detailed". Every request has unstated assumptions, boundaries, and acceptance criteria.

### Phase 1: Planning (`dip:deep-plan`)
- **Runbook**: Read instructions using `view_file` on `dip:deep-plan` runbook (resolve path: Available skills -> `~/.agents/skills/dip-deep-plan/SKILL.md` -> `.agents/skills/dip-deep-plan/SKILL.md` -> `skills/dip-deep-plan/SKILL.md`).
- **Input Check**: Inspect `.dip/plans/` for an existing plan matching the spec.
- If existing approved plan exists: skip directly to Phase 2.
- Otherwise:
  - Formulate draft plan with RALPLAN-DR framework.
  - Dispatch Architect and Critic subagents via `invoke_subagent` (`Model: "pro"`).
  - Iterate until unanimous consensus is recorded.
  - Yields `.dip/plans/plan-{slug}.md` (`Status: PENDING APPROVAL`).
  - **Approval Gate**: Prompt user for explicit approval to begin execution.

### Phase 2: Execution (`dip:execute`)
- **Runbook**: Read instructions using `view_file` on `dip:execute` runbook (resolve path: Available skills -> `~/.agents/skills/dip-execute/SKILL.md` -> `.agents/skills/dip-execute/SKILL.md` -> `skills/dip-execute/SKILL.md`).
- Read approved `.dip/plans/plan-{slug}.md`.
- Break plan into atomic work milestones.
- Dispatch implementation tasks to isolated executor subagents via `invoke_subagent` (`TypeName: "self"`, `Role: "Code Executor"`).
- Run independent components in parallel if `--parallel` is active.

### Phase 3: QA Cycling (`dip:verify`)
- **Runbook**: Read instructions using `view_file` on `dip:verify` runbook (resolve path: Available skills -> `~/.agents/skills/dip-verify/SKILL.md` -> `.agents/skills/dip-verify/SKILL.md` -> `skills/dip-verify/SKILL.md`).
- Dispatch QA Engineer subagent via `invoke_subagent` (`TypeName: "self"`, `Role: "QA Engineer"`).
- Run project build, lint, and test suites.
- If failures occur: diagnose and apply targeted fixes (up to 5 cycles).
- **Guardrail**: If the identical error signature occurs 3 times, abort and escalate to user.

### Phase 4: Validation (`dip:review`)
- **Runbook**: Read instructions using `view_file` on `dip:review` runbook (resolve path: Available skills -> `~/.agents/skills/dip-review/SKILL.md` -> `.agents/skills/dip-review/SKILL.md` -> `skills/dip-review/SKILL.md`).
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
