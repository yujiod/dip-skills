---
name: dip:deep-interview
description: Socratic deep interview with mathematical ambiguity gating before explicit execution approval
argument-hint: "[--quick|--standard|--deep] [--frontier] <idea or vague description>"
pipeline: [dip:deep-interview, dip:deep-plan]
handoff-policy: approval-required
handoff: .dip/specs/deep-interview-{slug}.md
level: 3
---

# Deep Interview (`dip:deep-interview`)

<Purpose>
Deep Interview implements Socratic questioning with mathematical ambiguity scoring. It replaces vague ideas with crystal-clear specifications by asking targeted questions that expose hidden assumptions, measuring clarity across weighted dimensions, and refusing to proceed until ambiguity drops below the resolved threshold. The output feeds into a gated pipeline: **dip:deep-interview → dip:deep-plan consensus deliberation → explicit approval → execution**, ensuring maximum clarity before any mutation begins.
</Purpose>

<Use_When>
- User has a vague idea or complex requirement needing thorough gathering before execution
- User says "deep interview", "interview me", "ask me questions", "don't assume", or "make sure you understand"
- User wants to avoid misunderstandings or wasted cycles in autonomous execution
- User desires mathematically gated clarity before committing to architectural design or code changes
</Use_When>

<Do_Not_Use_When>
- User provides a complete, specific specification with exact file paths, interfaces, and acceptance criteria -- execute or plan directly
- User asks for a quick single-file fix or trivial change
- User explicitly says "skip questions" or "just do it" -- write a pending specification artifact with recorded assumptions rather than guessing
</Do_Not_Use_When>

<Execution_Policy>
1. **No Cold Questions (Strict Sequence)**: Never invoke `ask_question` immediately on Turn 1 without prior codebase inspection and initial ambiguity score announcement.
2. **Interactive Questioning**: Use Antigravity's `ask_question` tool whenever interactive clarification is needed. Format questions clearly with recommended options.
3. **One Question at a Time**: Target the weakest clarity dimension in each round unless `--frontier` mode is explicitly requested.
4. **Round 0 Topology Gate**: Enumerate and confirm top-level components before running depth-first inquiry.
5. **Codebase Exploration First**: Use file viewing or search tools before asking questions about existing code (brownfield). Cite findings directly.
6. **Mathematical Ambiguity Scoring**: Score clarity after every user answer and display ambiguity transparently.
7. **No Mutation Before Approval**: Never edit source code, execute destructive commands, or commit changes during the interview.
8. **Artifact Discipline**: Save final specifications to `.dip/specs/deep-interview-{slug}.md`. Store state in `.dip/state/` if needed.
9. **User Language Match**: Conduct user-facing dialogue and questions in the language used by the user (defaulting to Japanese if addressed in Japanese), maintaining natural and professional phrasing.
10. **Subagent Delegation Architecture (Mandatory Separation of Exploration & Analysis)**:
    - **Lead Agent Orchestration**: Interactive UI modals (`ask_question`) are handled exclusively by the Lead agent in the main chat session.
    - **Explore Subagent ([`agents/dip-explore.md`](../../agents/dip-explore.md))**: In Phase 1 and brownfield investigations, dispatch `dip-explore` subagents via `invoke_subagent(TypeName: "research", Role: "Codebase Explorer", Model: "flash")` to inspect existing code patterns, dependencies, and prior specs before designing questions. In Frontier mode, dispatch `dip-explore` to resolve environmental facts without asking the user.
    - **Analyst Subagent ([`agents/dip-analyst.md`](../../agents/dip-analyst.md))**: In Phase 2, leverage `dip-analyst` (`invoke_subagent(TypeName: "research", Role: "Requirements Analyst", Model: "pro")`) to identify missing questions, undefined guardrails, scope creep risks, and unvalidated assumptions, sharpening Socratic questions and calculating dimensional clarity gaps.
    - **Critic Subagent ([`agents/dip-critic.md`](../../agents/dip-critic.md))**: On Round 3+ Challenge Perspectives (Skeptic / Contrarian), dispatch `dip-critic` (`invoke_subagent(TypeName: "research", Role: "Skeptic Critic", Model: "pro")`) to execute pre-mortems, test fragile assumptions, and challenge architectural orthodoxies before spec crystallization.
</Execution_Policy>

## Ambiguity Scoring Dimensions

Clarity is measured across 4 weighted dimensions (sum of weights = 1.0):

- **Goal Clarity ($W_{goal} = 0.30$)**: What is being built, why, what is the core value proposition, and what are explicit non-goals?
- **Constraints & Guardrails ($W_{constraints} = 0.25$)**: Technical boundaries, performance limits, security/compliance, compatibility, and forbidden approaches.
- **Acceptance Criteria ($W_{criteria} = 0.25$)**: How to verify success, edge cases, test expectations, failure modes, and measurable conditions.
- **Context & Environment ($W_{context} = 0.20$)**: Integration touchpoints, existing patterns, user personas, runtime conditions, and dependencies.

### Formula
$$\text{Clarity} = 0.30 \cdot C_{goal} + 0.25 \cdot C_{constraints} + 0.25 \cdot C_{criteria} + 0.20 \cdot C_{context}$$
$$\text{Ambiguity} = 1.0 - \text{Clarity}$$

- $C_d \in [0.0, 1.0]$: Dimension clarity score.
- Default threshold: $\text{Ambiguity} \le 0.20$ (Clarity $\ge 80\%$).
- Quick mode (`--quick`): $\text{Ambiguity} \le 0.35$.
- Deep mode (`--deep`): $\text{Ambiguity} \le 0.10$.

---

## Phases & Workflow

### Phase 0: Resolve Threshold & Setup
1. Determine threshold based on flags (`--quick`: 0.35, `--deep`: 0.10, default: 0.20).
2. Announce interview start to user:
   - Target threshold percentage
   - Input idea summary
   - Project type (Greenfield vs Brownfield)

### Phase 1: Context & Brownfield Analysis
1. Inspect workspace to check if this is brownfield (existing code, git repository) or greenfield.
2. If brownfield:
   - Dispatch `explore` subagent via `invoke_subagent(TypeName: "research", Role: "Codebase Explorer", Model: "flash")` to search relevant files, packages, architecture patterns, and prior specs/plans in `.dip/specs/` or `.dip/plans/`.
   - Incorporate findings into session context and cite repository facts directly instead of asking user to explain existing code.

### Round 0: Topology Enumeration Gate
1. Extract 1 to 6 top-level components or workstreams from the initial request and codebase context (informed by `explore` findings).
2. Present the candidate components to the user using `ask_question`:
   - "I identified {N} top-level components: [List]. Does this topology accurately capture the scope? Should anything be added, merged, or deferred?"
3. Lock the confirmed components into the session scope.

### Phase 2: Socratic Interview Loop
Repeat until $\text{Ambiguity} \le \text{Threshold}$ or user explicitly exits:
1. Identify the **weakest dimension** ($C_d$ with lowest score or largest information gap).
2. Formulate 1 targeted question to resolve the biggest uncertainty in that dimension. Optionally consult `analyst` subagent (`invoke_subagent(TypeName: "research", Role: "Requirements Analyst", Model: "pro")`) to extract unvalidated assumptions and testable acceptance criteria.
3. Present the question using `ask_question` with recommended options and direct response format.
4. Update clarity scores for each dimension upon receiving the response.
5. Display current ambiguity score and remaining gap:
   ```
   [Round {N}] Ambiguity: {Score}% (Target: ≤{Threshold}%) | Weakest: {Dimension}
   ```

#### Challenge Perspectives (Active on specific rounds):
- **Round 3+**: *Skeptic* - Dispatch `critic` subagent (`invoke_subagent(TypeName: "research", Role: "Skeptic Critic", Model: "pro")`) to challenge assumptions regarding performance bottlenecks, edge case failures, or user error via pre-mortem inquiry.
- **Round 5+**: *Minimalist* - Challenge whether components can be simplified, scoped down, or deferred.

#### Frontier Mode (`--frontier`):
If `--frontier` is active:
- Map dependencies as a design decision tree.
- Batch up to 4 non-dependent questions currently on the decision frontier per round using `ask_question`.
- When a frontier question requires environmental facts, dispatch `dip-explore` subagent ([`agents/dip-explore.md`](../../agents/dip-explore.md)) rather than querying the user.

### Phase 3: Crystallize Specification
When $\text{Ambiguity} \le \text{Threshold}$:
1. Generate the specification document at `.dip/specs/deep-interview-{slug}.md`.
2. Format:
   - **Title & Metadata**: Date, threshold reached, initial idea, scope topology.
   - **Executive Summary**: Core purpose, value, and scope.
   - **Explicit Non-Goals**: Clear negative constraints.
   - **Architectural & Technical Constraints**: Standards, performance, security.
   - **Detailed Component Specifications**: Requirements for each confirmed component.
   - **Acceptance Criteria & Test Matrix**: Concrete verification checklist.
   - **Status**: `Pending Approval`.

### Phase 4: Handoff to Deliberation & Planning
1. Notify the user that the specification is finalized and display its path.
2. Present a structured proposal to proceed to consensus planning:
   - "Specification finalized at `.dip/specs/deep-interview-{slug}.md`. Would you like to proceed with `dip:deep-plan` to deliberate and structure the implementation plan?"
3. Await user confirmation before initiating `dip:deep-plan`.
