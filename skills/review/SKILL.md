---
name: dip:review
description: Multi-perspective validation by independent Architect, Security, and Code Reviewer subagents
argument-hint: "[--rounds <N>] <target scope or diff>"
pipeline: [dip:review, complete]
handoff-policy: approval-required
handoff: .dip/reviews/review-{slug}.md
level: 4
---

# Review (`dip:review`)

<Purpose>
`dip:review` evaluates finished, verified work for architectural soundness, security integrity, code quality, and slop elimination before it ships. Reviewers NEVER author the changes they judge. It enforces 3-way cognitive diversity by dispatching independent subagents for Architect, Security, and Code reviews in parallel.
</Purpose>

<Use_When>
- Implementation and QA verification (`dip:verify`) have completed
- Final validation before deployment, pull request, or user delivery
- Autopilot enters Phase 4 (Validation)
</Use_When>

<Do_Not_Use_When>
- Code does not yet build or tests are failing -- use `dip:verify` first
- Work is in early exploration -- use `dip:deep-interview` or `dip:deep-plan`
</Do_Not_Use_When>

<Execution_Policy>
1. **Mandatory Subagent Separation (Strictly No Self-Review)**:
   - Self-review within the lead or executor context is **strictly prohibited**.
   - The primary agent coordinates reviews and aggregates verdicts.
   - Three independent subagents **MUST** be launched in parallel via `invoke_subagent` (`Model: "pro"`):
     - **Architect Reviewer**: Verifies functional completeness, design adherence, and boundaries.
     - **Security Reviewer**: Verifies vulnerabilities, authentication, authorization, injection, and secrets.
     - **Code Reviewer**: Verifies cleanliness, maintainability, tests, and removes AI slop/redundancy.
2. **Unanimous Consensus Gate (All Must Approve)**:
   - Every reviewer must explicitly issue `APPROVE` before code is accepted.
   - Any `REQUEST_CHANGES` verdict blocks completion.
3. **Bounded Remediation Loop**:
   - Up to 3 validation rounds (`maxValidationRounds = 3`).
   - If changes are requested, a remediation subagent applies the targeted fixes, passes `dip:verify`, and re-submits for re-review.
4. **Escalation & Stop Condition**:
   - If consensus is not reached after 3 rounds, **STOP IMMEDIATELY**.
   - Present the conflicting requirements or persistent issues directly to the user.
5. **Artifact Output**:
   - Final review summary must be recorded in `.dip/reviews/review-{slug}.md`.
</Execution_Policy>

---

## The 3-Perspective Reviewers

```
              [Verified Codebase & Diff]
                          │
          ┌───────────────┼───────────────┐
          │ (invoke)      │ (invoke)      │ (invoke)
          v               v               v
   [1. Architect]   [2. Security]   [3. Code Reviewer]
   - Completeness   - OWASP Top 10  - Simplicity
   - Plan alignment - Auth/Secrets  - Test coverage
   - Boundary leaks - Data sanitize - Anti-Slop
          │               │               │
          └───────────────┼───────────────┘
                          │
                          v
            [Consensus Aggregator Gate]
            ├─ All Approved? ──► Write .dip/reviews/review-{slug}.md -> Complete
            └─ Any Rejection? ──► Remediate & Re-verify (Max 3 rounds)
```

### 1. Architect Reviewer Subagent
- Dispatch: `invoke_subagent` (`TypeName: "research"`, `Role: "Architect Reviewer"`, `Model: "pro"`).
- Criteria: Does the code fulfill every requirement in the specification? Are architectural boundaries preserved? Any unauthorized scope creep?

### 2. Security Reviewer Subagent
- Dispatch: `invoke_subagent` (`TypeName: "research"`, `Role: "Security Reviewer"`, `Model: "pro"`).
- Criteria: Check input validation, credential handling, token expiration, injection vectors, file traversal, and sensitive data leakage.

### 3. Code Reviewer Subagent
- Dispatch: `invoke_subagent` (`TypeName: "research"`, `Role: "Code Reviewer"`, `Model: "pro"`).
- Criteria: Code style, maintainability, unnecessary boilerplate/AI-slop, edge case handling, reuse of existing project helpers.

---

## Workflow

### Step 1: Collect Context & Diffs
- Generate git diff or file snapshots of all modifications made during execution.
- Fetch original requirements from `.dip/specs/` or plan from `.dip/plans/`.

### Step 2: Parallel Review Dispatch
- Dispatch the 3 reviewer subagents in parallel with the diff and criteria.
- Receive findings, severity rankings, and explicit verdicts (`APPROVE` or `REQUEST_CHANGES`).

### Step 3: Synthesis & Remediation Gate
- If all 3 approve:
  - Generate `.dip/reviews/review-{slug}.md` with status `APPROVED`.
  - Signal readiness for completion or pull request.
- If any reviewer requests changes:
  - Group findings by severity (Critical, High, Medium, Low).
  - Launch executor subagent to resolve actionable findings.
  - Re-verify with `dip:verify`.
  - Re-submit updated diff to reviewers (Round N+1).

### Step 4: Final Sign-off
- Output verdict summary to user with link to `.dip/reviews/review-{slug}.md`.
