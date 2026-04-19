---
description: "Orchestrate a strict DESIGN > RED > GREEN > REFACTOR > QUALITY Test-Driven Development (TDD) loop with specialized agents and Human-in-the-Loop gates. Stack-agnostic — reads your CLAUDE.md for project conventions."
argument-hint: "Task description, issue reference, or Prompt Contract"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep", "Agent"]
---

## User Input

$ARGUMENTS

You **MUST** consider the user input before proceeding (if not empty).

## Available Agents

| Agent | Role | Model | Tools | Phase |
|-------|------|-------|-------|-------|
| `@ui-designer` | UI/UX specifications, component design, accessibility | Sonnet | Read-only | DESIGN |
| `@api-designer` | API contracts, endpoint schemas, error formats | Sonnet | Read-only | DESIGN |
| `@infra-designer` | Infrastructure specs, CI/CD pipelines, containers | Sonnet | Read + Write + Bash | DESIGN |
| `@tester` | Failing tests (RED phase) | Sonnet | Read + Write + Bash | RED |
| `@implementer` | Minimum code to pass tests + approved refactoring | Sonnet | Read + Write + Bash | GREEN + REFACTOR |
| `@architect` | Audit implementation, produce refactoring plans | Opus | Read-only | REFACTOR audit |
| `@code-reviewer` | Code quality review | Opus | Read + Bash | QUALITY |
| `@security-auditor` | Security vulnerability scan | Opus | Read + Bash | QUALITY |

## Flow

```
Phase 1: ANALYSIS        Parse task -> identify affected domains
Phase 2: DESIGN          @ui-designer / @api-designer / @infra-designer (optional)
Phase 3: RED             @tester writes failing tests — Gate: tests FAIL
Phase 4: GREEN           @implementer writes minimum code — Gate: tests PASS
Phase 5: REFACTOR AUDIT  @architect audits code — HUMAN GATE: owner approves plan
Phase 6: REFACTOR APPLY  @implementer applies approved plan — Gate: tests PASS
Phase 7: QUALITY CHECKS  @code-reviewer + @security-auditor (parallel)
Phase 8: COMPLETION      Summary report
```

## Detailed Steps

### 1. Analysis & Domain Detection

Parse `$ARGUMENTS` and read the project to identify:

- **Project constitution:** Read CLAUDE.md (or equivalent). This is the supreme authority
  for all agents. If it does not exist, warn and continue with best practices.
- **Affected domains:** Examine the project structure and task description to determine
  which areas of the codebase will be touched.
- **Domain-to-agent mapping:** Determine which DESIGN agents are needed:

| If the task involves... | Invoke in DESIGN phase |
|------------------------|----------------------|
| UI screens, components, layouts, animations, styling | `@ui-designer` |
| New or changed API endpoints, service contracts | `@api-designer` |
| Both UI and API changes | `@ui-designer` AND `@api-designer` (sequentially) |
| Cloud resources, IaC, CI/CD pipelines, Docker | `@infra-designer` |
| Pure backend/library logic (no new interfaces) | Skip DESIGN phase entirely |

### 2. DESIGN Phase (Optional — one or more agents)

**If UI/UX work -> delegate to `@ui-designer`:**
- Handoff: Product requirements + relevant existing component code
- Expected outputs: `<design_rationale>`, `<wireframe>`, `<specifications>`, `<test_constraints>`
- Gate: Verify specifications are actionable — component structure, styling tokens, animation
  parameters, and accessibility labels are defined.

**If API contract work -> delegate to `@api-designer`:**
- Handoff: Feature requirements + existing service interfaces
- Expected outputs: Endpoint specs (method, path, request/response schemas, error codes)
- Gate: Verify endpoint contracts are complete with auth requirements and error formats.

**If infrastructure work -> delegate to `@infra-designer`:**
- Handoff: Resource or pipeline requirements
- Expected outputs: IaC specs, pipeline configs, or container definitions
- Gate: Configurations validated. **HUMAN GATE if apply/deploy is needed.**

### 3. RED Phase (Delegation: `@tester`)

Handoff the RED contract to `@tester`, including:
- The DESIGN phase outputs (test_constraints, API specs) if applicable
- The feature requirements from `$ARGUMENTS`
- The project's CLAUDE.md for test conventions

Constraints:
- The agent must ONLY write failing tests. NO implementation code.

Gate: Run tests and verify they FAIL.

### 4. GREEN Phase (Delegation: `@implementer`)

Handoff the GREEN contract to `@implementer`, including:
- The failing tests from Step 3
- The DESIGN specs (component specifications, endpoint contracts) if applicable
- The project's CLAUDE.md for coding conventions

Constraints:
- Write the MINIMUM code to make tests pass
- Do NOT modify tests to make them pass

Gate: Run tests and verify ALL PASS.

### 5. REFACTOR Phase — Audit & Proposal (Delegation: `@architect`)

Handoff the REFACTOR audit to `@architect`.

Constraints:
- Audit the GREEN implementation against the CLAUDE.md constitution
- Generate a strict **Refactoring Plan** with the prescribed format
- Apply the REFACTOR Scope Rules below when categorising findings

#### REFACTOR Scope Rules

The REFACTOR phase produces fixes, not follow-up issues. Every finding the
architect surfaces MUST be categorised as one of:

- **APPLY (inline)** — the finding can be fixed in < 30 min AND touches files
  already in the current change set. Include it in the plan so the implementer
  fixes it now. Do NOT recommend a follow-up issue for it.
- **OUT OF SCOPE (skip)** — the finding is in a file outside the current change
  set. Note it as "out of scope (untouched file)" in the audit report, then
  drop it. Do not file, do not defer, do not track. The next PR that modifies
  that file is where it belongs.
- **STYLE PREFERENCE (skip)** — a stylistic suggestion with no correctness,
  security, or performance impact (e.g. "switch could be a map",
  "could use `slices.Contains`"). Skip permanently — do not file, do not defer,
  do not track.
- **SPEC GAP (pause)** — the finding reveals a genuine requirements gap
  (missing requirement, contradiction, undecided security policy). STOP and
  surface to the owner. If the consumer project uses specflow, route through
  its `/specflow:specify` → `/specflow:contract` → `/specflow:plan` →
  `/specflow:publish` pipeline. Do NOT create standalone GitHub issues for
  spec gaps.

The architect's audit report MUST end with:
`New issues recommended: [count]` — target is 0. If count > 0, each MUST
include a justification for why it cannot be fixed inline in the current PR.

**CRITICAL GATE (Human-in-the-Loop):**
Stop here. Present the Refactoring Plan to the user. Ask which changes to
apply, modify, or reject.

**Do NOT proceed to Step 6 until the user provides explicit approval.**

### 6. REFACTOR Phase — Apply (Delegation: `@implementer`)

*Only run AFTER receiving user approval from Step 5.*

Handoff the user-approved Refactoring Plan to `@implementer`.

Constraints:
- Apply ONLY the approved changes
- Do NOT alter external behavior
- Do NOT break tests

Gate: Run FULL test suite and verify ALL still PASS.

### 7. Quality Checks (Parallel Delegation)

After REFACTOR apply passes, run these two agents **in parallel**:

**`@code-reviewer` — Code Quality Review:**
- Reads all files changed across the entire cycle
- Applies the full review checklist from its definition
- Produces a structured review report
- Does NOT modify any files

**`@security-auditor` — Security Scan:**
- Scans all changed files for vulnerabilities
- Applies OWASP checks + project-specific security rules from CLAUDE.md
- Produces a security audit report
- Does NOT modify any files

Gate: Both agents must report no Critical or High severity issues.
- If either reports Critical/High -> **STOP. Present findings to owner. Fix before proceeding.**
- If only Medium/Low/Info findings -> note them in the completion report, continue to Step 8.

### 8. Completion Reporting

Output a summary of the Triad loop:

```markdown
## Triad Cycle Complete — [Feature/Task Name]

### Phases Run
- [ ] DESIGN: @ui-designer / @api-designer / @infra-designer / skipped
- [ ] RED: @tester — N tests written
- [ ] GREEN: @implementer — N files created/modified
- [ ] REFACTOR: @architect audit -> owner approved -> @implementer applied
- [ ] CODE REVIEW: @code-reviewer — [PASS/ISSUES]
- [ ] SECURITY: @security-auditor — [PASS/ISSUES]

### Files Modified
[Grouped by directory/module]

### Test Results
- Framework: [test runner] — N/N passed, coverage: X%

### Open Items
- [Any Medium/Low findings to track]

### Scope Accounting
- New issues recommended: [count — target 0]
- Justifications: [one line per recommended issue, or "N/A — all findings fixed inline or skipped per Scope Discipline rules"]
- Vendor/spec conflicts flagged: [none, or describe each]
```

## Behavior Rules

1. **Strict Sequencing:** Never advance to the next phase until the current phase's Gate is passed.
2. **Interactive Pause:** Strictly respect the human gate at Step 5. Do not simulate or assume approval. If the code reviewer or security auditor report Critical/High issues in Step 7, stop and present to owner.
3. **Role Isolation:**
   - `@ui-designer` MUST NOT write implementation code
   - `@api-designer` MUST NOT write implementation code
   - `@infra-designer` MUST NOT write application code
   - `@tester` MUST NOT write feature code
   - `@implementer` MUST NOT rewrite tests to make them pass
   - `@architect` MUST NOT write refactored code
   - `@code-reviewer` MUST NOT modify files
   - `@security-auditor` MUST NOT modify files
4. **Constitution as Authority:** CLAUDE.md is loaded into every agent's context. It is the supreme authority for all conventions, patterns, and rules.
5. **No Nesting:** Subagents cannot spawn other subagents. Each agent completes its work and returns results to the orchestrator.
6. **Cross-Cutting Tasks:** A single task touching multiple domains runs through ONE triad loop — do not split into separate loops.
7. **Scope Discipline:** Every triad loop is bounded by the files it touches and the requirements it was given.
   - **Fix inline, don't defer.** Findings < 30 min that touch in-scope files are fixed in the current loop, not filed as issues.
   - **Zero new issues is the target.** Every follow-up issue a triad loop produces is scope creep until justified. The completion report MUST state how many new issues were recommended and why each was unavoidable.
   - **Search before creating.** Before recommending ANY new issue, the orchestrator (or owner) MUST run `gh issue list --search "<keywords>" --state open` and verify no existing issue covers the concern. Duplicate issues are a defect.
   - **Style preferences are not findings.** Suggestions with no correctness, security, or performance impact (switch vs map, naming nits in untouched files, `slices.Contains` vs explicit loop) are SKIPPED — not filed, not deferred, not tracked.
   - **Don't touch what you didn't break.** Findings in files outside the current change set are out of scope. They belong to the next PR that modifies those files.
   - **Spec gaps use specflow, not issues.** If the audit reveals a genuine spec gap, pause and surface to the owner. When the consumer project uses specflow, route through `/specflow:specify` → `/specflow:contract` → `/specflow:plan` → `/specflow:publish`. Never create a standalone GitHub issue for a spec gap.
   - **Vendor docs over contract specs on conflict.** When a provider's current documentation conflicts with the contract's FORMAT or patterns, follow the vendor's recommended approach and flag the discrepancy in the completion report. Check live documentation before assuming a referenced pattern is still current.
