---
name: verify
description: Rigorous QA cycle executing builds, linters, tests, and automated fix loops with dedicated QA subagents
argument-hint: "[--max-cycles <N>] <scope or test commands>"
pipeline: [verify, deep-review]
handoff-policy: direct
level: 4
---

# Verify (`verify`)

<Purpose>
`verify` turns vague "it should work" assumptions into concrete, observable evidence. It orchestrates end-to-end quality assurance (build, lint, typecheck, unit tests, integration tests) and manages bounded automated bug-fixing cycles via dedicated QA subagents. No implementation claims completion without passing this verification gate.
</Purpose>

<Use_When>
- Implementation has completed under `execute` or manual coding
- User requests QA verification, test execution, or build confidence
- Autopilot enters Phase 3 (QA)
</Use_When>

<Do_Not_Use_When>
- Implementation is not yet in place -- use `execute` first
- High-level architecture or requirements are missing -- use `deep-interview` or `deep-plan`
</Do_Not_Use_When>

<Execution_Policy>
1. **Mandatory Subagent Separation (Strictly No Self-Validation)**:
   - Verification runs and automated remediation **MUST** be performed by dedicated QA subagents via `invoke_subagent` using the corresponding agents defined in `agents/`:
     - **QA Tester** (`TypeName: "qa-tester"`, `Role: "QA Engineer"`): Executes builds, linters, and test suites adhering to [`agents/qa-tester.md`](../../agents/qa-tester.md).
     - **Test Engineer** (`TypeName: "test-engineer"`, `Role: "Test Engineer"`): Designs test strategy gaps, integration/e2e coverage, and missing tests ([`agents/test-engineer.md`](../../agents/test-engineer.md)).
     - **Debugger / Tracer** (`TypeName: "debugger"`, `Role: "Debugger"` or `TypeName: "tracer"`, `Role: "Causal Tracer"`): Performs failure analysis, root-cause diagnosis, and minimal targeted fixes ([`agents/debugger.md`](../../agents/debugger.md), [`agents/tracer.md`](../../agents/tracer.md)).
     - **Verifier** (`TypeName: "verifier"`, `Role: "Verification Auditor"`): Audits final acceptance criteria sign-off and empirical verification evidence ([`agents/verifier.md`](../../agents/verifier.md)).
   - Primary agent coordinates execution cycles and monitors guardrails.
2. **Deterministic Verification Sequence**:
   1. Build / Compilation check
   2. Lint & Typecheck
   3. Existing unit / integration tests
   4. Newly introduced feature tests
3. **Bounded Remediation Loop**:
   - Up to 5 QA cycles allowed by default (`maxQaCycles = 5`).
   - If tests fail, the QA subagent diagnoses the root cause (using `debugger`, `TypeName: "debugger"`), applies a minimal targeted fix, and re-runs tests.
4. **Escalation & Stop Condition**:
   - **Identical Error Guard**: If the exact same failure persists across 3 consecutive cycles, **STOP IMMEDIATELY**. Report the fundamental blocker to the user with full diagnostic evidence.
   - If 5 cycles are exhausted without full pass, stop and prompt user.
5. **No Claims Without Evidence**:
   - A task is NEVER declared complete without fresh command output proving successful test and build execution verified by `verifier`.
</Execution_Policy>

---

## Workflow

```
[Implemented Code]
        │
        v
   [1. QA Plan] ──────────> Identify build, lint, and test commands
        │
        v
   [2. QA Subagent] ──────> invoke_subagent(TypeName="qa-tester", Role="QA Engineer")
        │                    Execute build -> lint -> test suite
        │
   ┌────┴──────────────────────────┐
   │ Check Outcome                 │
   │                               │
   ├─► All Passed ─────────────────┼───► [Proceed to deep-review]
   │                               │
   └─► Failures Detected ──────────┘
             │
             ├─ Same error >= 3 times? ──► [ESCALATE & STOP]
             ├─ Cycles > 5? ─────────────► [ESCALATE & STOP]
             └─ Otherwise ───────────────► [Fix & Loop to Step 2]
```

### Step 1: Verification Scope Detection
1. Discover project test harnesses (e.g. `npm test`, `pytest`, `cargo test`, `go test`, `vitest`).
2. Identify lint and build targets.
3. Formulate the verification matrix based on modified files and the original acceptance criteria.

### Step 2: QA Dispatch (`invoke_subagent`)
Launch a QA subagent:
- `TypeName`: `"qa-tester"`
- `Role`: `"QA Engineer"`
- `Prompt`: Run build, lint, and test commands; inspect failure stack traces; isolate causes.
*(Optional / Failure handling)*: If test strategy/harness gaps are identified, dispatch `TypeName: "test-engineer"`. If test failures require deep diagnosis, dispatch `TypeName: "debugger"` or `TypeName: "tracer"`. For final criteria verification, dispatch `TypeName: "verifier"`.

### Step 3: Diagnostic & Fix Cycle
- If any check fails:
  - Inspect error signature. Compare against previous cycle signatures.
  - If identical error repeats 3 times: abort cycle, document the impasse, and alert the user.
  - If distinct error: subagent applies minimal fix, confirms syntax, and repeats tests.

### Step 4: Verification Artifact & Handoff
- Collect clean pass logs.
- Record verification outcome in `.dip/state/qa-evidence.json`.
- Hand off verified codebase directly to `deep-review`.
