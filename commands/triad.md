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
