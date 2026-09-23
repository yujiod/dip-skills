---
name: dip:execute
description: Carry an approved plan through to working, verified code using isolated executor subagents
argument-hint: "[--parallel] <plan path or task description>"
pipeline: [dip:execute, dip:verify]
handoff-policy: direct
level: 4
---

# Execute (`dip:execute`)

<Purpose>
`dip:execute` carries an approved plan (from `dip:deep-plan` or explicit user task) through to working implementation. It enforces strict separation of concerns by dispatching implementation work to isolated executor subagents via `invoke_subagent`. It translates architectural milestones into atomic, verifiable code modifications without noisy or unstructured modifications.
</Purpose>

<Use_When>
- An approved implementation plan exists in `.dip/plans/plan-{slug}.md`
- User explicitly requests execution of an agreed-upon architecture or specification
- Autopilot enters Phase 2 (Execution)
</Use_When>

<Do_Not_Use_When>
- Requirements are still vague or unclarified -- use `dip:deep-interview` first
- Architecture is undecided or high-risk changes lack consensus -- use `dip:deep-plan` first
- Plan is marked `PENDING APPROVAL` without explicit user consent
</Do_Not_Use_When>

<Execution_Policy>
1. **Mandatory Subagent Separation (Strictly No Direct In-Place Implementation)**:
   - Primary agent acts as orchestrator / team lead.
   - Code mutations **MUST** be delegated to isolated executor subagents using `invoke_subagent` (`TypeName: "self"`, `Role: "Code Executor"`) adhering to [`agents/dip-executor.md`](../../agents/dip-executor.md).
   - The primary agent must never mutate source files directly when running `dip:execute`; all mutations are made by dispatched executor subagents.
   - Code refinement, nesting elimination, and anti-slop cleaning may be dispatched to `dip-code-simplifier` ([`agents/dip-code-simplifier.md`](../../agents/dip-code-simplifier.md)).
   - Version control operations and clean commit splitting are managed via `dip-git-master` ([`agents/dip-git-master.md`](../../agents/dip-git-master.md)).
2. **Phase-by-Phase Discipline**:
   - Deconstruct the plan into bounded, atomic milestones.
   - Independent work units may be dispatched to multiple parallel executor subagents (`--parallel`).
   - Dependent units must execute strictly sequentially.
3. **Continuous Local Sanity Check**:
   - Each executor subagent must ensure modified files compile, parse, or typecheck before reporting completion back to the lead.
4. **Traceable State Tracking**:
   - Record progress and completed milestones in `.dip/state/execute-progress.json`.
5. **Clean Handoff to Verification**:
   - Upon completing plan milestones, transition directly to `dip:verify` for full QA and test cycles.
</Execution_Policy>

---

## Workflow

```
[Approved Plan / Task]
         │
         v
[1. Task Breakdown] ───────> Identify atomic units & dependency graph
         │
         v
[2. Dispatch Executors] ───> invoke_subagent(TypeName="self", Role="Code Executor")
         │                   (Parallel for independent tasks, sequential for dependents)
         v
[3. Milestone Sanity] ─────> Verify local syntax/type validity per unit
         │
         v
[4. Completion Report] ────> Summarize changed files & forward to dip:verify
```

### Step 1: Plan Ingestion & Work Breakdown
1. Read the approved plan (`.dip/plans/plan-{slug}.md` or input task).
2. Extract concrete tasks, modified file targets, and acceptance criteria.
3. Determine task dependencies and identify parallelizable chunks.

### Step 2: Executor Dispatch (`invoke_subagent`)
For each task unit:
- Dispatch a subagent via `invoke_subagent`:
  - `TypeName`: `"self"`
  - `Role`: `"Code Executor: [Task Name]"`
  - `Prompt`: Include target file paths, interface specs, exact diff guidelines, and minimal mutation requirements.
- Wait for subagent completion and collect modified file summaries.

### Step 3: Incremental Sanity Checks
- Confirm no merge conflicts or broken imports across dispatched units.
- Ensure all targeted files from the plan step were touched and properly updated.

### Step 4: Execution Summary & Handoff
- Generate an execution summary:
  - List of modified files
  - Key architectural changes completed
  - Any deferred items or caveats
- Immediately hand off to `dip:verify` for automated test execution and bug-fix cycling.
