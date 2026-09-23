---
name: dip:deep-plan
description: Consensus planning with Planner, Architect, and Critic agents using RALPLAN-DR structured deliberation
argument-hint: "[--interactive] [--deliberate] <task description or spec path>"
pipeline: [dip:deep-plan, dip:execute]
handoff-policy: approval-required
handoff: .dip/plans/plan-{slug}.md
level: 4
---

# Deep Plan (`dip:deep-plan`)

<Purpose>
Deep Plan triggers iterative, consensus-driven architecture and implementation planning among **Planner**, **Architect**, and **Critic** roles until mutual agreement is reached. It employs **RALPLAN-DR structured deliberation** (short mode by default, deliberate mode for high-risk work) to produce robust, battle-tested execution plans before any source code mutation begins.
</Purpose>

<Use_When>
- User provides a specification from `dip:deep-interview` or an architectural challenge
- User wants consensus planning, structured deliberation, or multi-agent review
- Task involves high-risk changes (security/auth, schema migration, destructive refactor, public API change)
- User wants to evaluate viable alternatives, trade-offs, and pre-mortems before coding
</Use_When>

<Do_Not_Use_When>
- User asks for a trivial single-line change or routine bugfix -- plan directly without multi-role deliberation
- User has already approved a plan and requests execution -- proceed to execution phase
</Do_Not_Use_When>

<Planning_Execution_Boundary>
`dip:deep-plan` is strictly a planning and architecture skill.
- It MUST NOT edit application source code, run mutating shell commands, commit, push, or execute changes.
- All outputs must be saved to `.dip/plans/plan-{slug}.md` with status `pending approval`.
- Execution can only begin after explicit user approval of the finalized plan.
</Planning_Execution_Boundary>

<Execution_Policy>
1. **Mandatory Subagent Separation (Strictly No Self-Agreement & Full Tri-Agent Separation)**:
   - Self-agreement, self-drafting, or persona play within the primary agent context is **strictly prohibited**.
   - The primary agent acts as **Orchestrator / Facilitator** and does NOT draft or mutate plans directly.
   - Plan drafting and plan revisions **MUST** be delegated to the **Planner** subagent via `invoke_subagent` using [`agents/dip-planner.md`](../../agents/dip-planner.md):
     - `TypeName: "dip-planner"`, `Role: "Work Planner"`, `Model: "pro"`
   - Codebase facts and existing architecture are investigated via `dip-explore` subagent via `invoke_subagent(TypeName: "dip-explore", Role: "Codebase Explorer", Model: "flash")` ([`agents/dip-explore.md`](../../agents/dip-explore.md)).
   - External library/API specifications are looked up via `dip-document-specialist` via `invoke_subagent(TypeName: "dip-document-specialist", Role: "Doc Specialist", Model: "flash")` ([`agents/dip-document-specialist.md`](../../agents/dip-document-specialist.md)).
   - The **Architect** ([`agents/dip-architect.md`](../../agents/dip-architect.md)) and **Critic** ([`agents/dip-critic.md`](../../agents/dip-critic.md)) roles **MUST** be dispatched as independent, isolated subagents using `invoke_subagent`:
     - Architect: `TypeName: "dip-architect"`, `Role: "System Architect"`, `Model: "pro"`
     - Critic: `TypeName: "dip-critic"`, `Role: "Critical Reviewer"`, `Model: "pro"`
   - Consensus requires explicit approval from both Architect and Critic subagents on the plan produced and maintained by Planner. The primary agent must never draft, revise, or approve plans on behalf of any subagent.
2. **RALPLAN-DR Framework**:
   - **R**equirements & Principles (3-5 overarching engineering tenets)
   - **A**lternatives & Options (>=2 viable options analyzed with explicit trade-offs)
   - **L**eading Decision Drivers (top 3 factors deciding the chosen path)
   - **P**re-mortem (required in deliberate mode: 3 failure scenarios & defenses)
   - **A**cceptance & Test Plan (unit, integration, e2e, observability)
   - **N**ext Steps (milestones and atomic implementation tasks)
3. **Artifact Path**: Plans must be written to `.dip/plans/plan-{slug}.md`.
4. **User Language Match**: Conduct user-facing dialogue and questions in the language used by the user (defaulting to Japanese if addressed in Japanese).
</Execution_Policy>

---

## Flags & Modes

- `--interactive`: Prompts user at key checkpoints (draft plan review and final consensus sign-off) via `ask_question`. When omitted, runs automated deliberation up to final plan generation, stopping at `pending approval`.
- `--deliberate`: Enforces deliberate mode. Mandates 3 pre-mortem failure scenarios and comprehensive test strategy (unit, integration, e2e, observability). Auto-activates if the task involves security, migrations, destructive changes, or breaking API changes.

---

## The Consensus Deliberation Workflow

```
[Input Spec / Task]
        │
        v
   [1. Dispatch Planner] ────> invoke_subagent(TypeName="dip-planner", Role="Work Planner")
        │                      Generates Draft Plan & RALPLAN-DR Summary (.dip/plans/plan-{slug}.md)
        ├──────────────────── [Optional: User Feedback if --interactive]
        │
        v
   [2. Dispatch Architect] ──> invoke_subagent(TypeName="dip-architect", Role="System Architect")
        │                      Evaluates Architectural Soundness & Tradeoffs (Steelman antithesis)
        v
   [3. Dispatch Critic] ─────> invoke_subagent(TypeName="dip-critic", Role="Critical Reviewer")
        │                      Evaluates Principle Consistency & Failure Modes
        │
        ├─ Consensus Reached?
        │    ├─ No  ───> [Refinement Loop: Dispatch dip-planner with feedback (Max 3 rounds)]
        │    └─ Yes ───> [Proceed to Step 5]
        v
   [4. Final Plan Artifact] ──> .dip/plans/plan-{slug}.md (Status: PENDING APPROVAL)
        │
        v
   [5. Explicit Approval Prompt]
```

### Step 1: Dispatch Planner Draft (`invoke_subagent`)
The primary agent dispatches the **Planner** via `invoke_subagent`:
- `TypeName`: `"dip-planner"`
- `Role`: `"Work Planner"`
- `Model`: `"pro"`
- `Prompt`: Provide input specification (from `.dip/specs/` or user input), context, target flags (`--deliberate`), and target output path (`.dip/plans/plan-{slug}.md`). Instruct Planner to inspect workspace context and formulate the initial draft plan along with a compact **RALPLAN-DR summary**:
  1. **Core Principles**: 3 to 5 non-negotiable architectural principles governing this change.
  2. **Decision Drivers**: Top 3 engineering drivers (e.g., latency, backward compatibility, simplicity).
  3. **Viable Options**: At least 2 distinct technical options with bounded pros and cons. If only 1 option is selected, provide explicit justification invalidating alternatives.
  4. **Implementation Breakdown**: Phased breakdown of work with file-level targets.
  5. *(If `--deliberate`)*: Pre-mortem scenarios and expanded test matrix.
  The Planner writes the initial draft to `.dip/plans/plan-{slug}.md` and returns the RALPLAN-DR summary.

### Step 2: User Checkpoint (Interactive Mode Only)
If `--interactive` is enabled, present the draft plan and RALPLAN-DR summary to the user using `ask_question`:
- Options: `Proceed to review`, `Request adjustments`, `Cancel`.
- If user requests adjustments: dispatch `dip-planner` (`Role: "Work Planner Refinement"`) with user feedback to revise `.dip/plans/plan-{slug}.md` before proceeding to review.

### Step 3: Architect Review (Dispatched Subagent)
Launch the Architect using `invoke_subagent` (`TypeName: "dip-architect"`, `Role: "System Architect"`, `Model: "pro"`).
The Architect reviews the snapshot without mutating it:
- **Steelman Antithesis**: Formulate the strongest possible argument against the chosen approach.
- **Trade-off Tensions**: Highlight architectural friction points (e.g., memory vs throughput, abstraction vs speed).
- **Synthesis Recommendation**: Propose reconciliations or confirm the draft's viability.
- *(In deliberate mode)*: Flag any violations of the 3-5 core principles.
- Returns explicit verdict: `APPROVE` or `REQUEST_CHANGES` with actionable reasons.

### Step 4: Critic Review (Dispatched Subagent)
Launch the Critic using `invoke_subagent` (`TypeName: "dip-critic"`, `Role: "Critical Reviewer"`, `Model: "pro"`).
The Critic acts as the gatekeeper of completeness and risk mitigation:
- Enforce option-principle consistency.
- Verify that every risk identified by the Architect has a concrete mitigation.
- Ensure acceptance criteria are unambiguous and testable.
- *(In deliberate mode)*: Reject any plan lacking thorough pre-mortem failure scenarios or test coverage.
- Returns explicit verdict: `APPROVE` or `REQUEST_CHANGES` with actionable reasons.

### Consensus & Iteration Gate (Planner Refinement Loop)
- If Architect and Critic provide approvals or minor remarks, consensus is reached. Proceed to Step 5.
- If major blockers, rejection criteria, or `REQUEST_CHANGES` are returned by either Architect or Critic:
  - Do NOT modify the plan directly in the primary agent.
  - Forward the Architect and Critic feedback to the **Planner** via `invoke_subagent`:
    - `TypeName`: `"dip-planner"`
    - `Role`: `"Work Planner Refinement"`
    - `Model`: `"pro"`
    - `Prompt`: Include the current plan path, Architect review findings, and Critic review findings. Instruct Planner to revise `.dip/plans/plan-{slug}.md` to address all critiques and update the RALPLAN-DR summary and ADR.
  - Re-run Step 3 and Step 4 with the revised plan.
  - Loop for up to 3 refinement rounds. If consensus fails after 3 rounds, halt and escalate unresolved tensions to the user.

### Step 5: Final Plan Output & Approval Gate
1. The agreed-upon plan is finalized at `.dip/plans/plan-{slug}.md`.
2. Plan Header must state:
   ```markdown
   # Implementation Plan: {Title}
   Status: PENDING APPROVAL
   Deliberation Mode: Standard | Deliberate
   Consensus Rounds: {N}
   ```
3. Prompt user:
   "Consensus reached between Planner, Architect, and Critic. Plan recorded at `.dip/plans/plan-{slug}.md`. Review the plan and approve to proceed with execution."
