# Triad Plugin Design Spec

**Date:** 2026-04-06
**Repo:** batidiane/triad
**Status:** Approved

## Summary

A standalone Claude Code plugin that provides a multi-agent TDD orchestrator following
DESIGN > RED > GREEN > REFACTOR > QUALITY phases. Generic and stack-agnostic — agents
read the project's CLAUDE.md for stack-specific conventions. Designed to integrate with
specflow (`/specflow:implement` invokes `/triad` if available) but works standalone.

## Plugin Structure

```
triad/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── commands/
│   └── triad.md
├── agents/
│   ├── ui-designer.md
│   ├── api-designer.md
│   ├── infra-designer.md
│   ├── tester.md
│   ├── implementer.md
│   ├── architect.md
│   ├── code-reviewer.md
│   └── security-auditor.md
├── LICENSE
├── package.json
└── README.md
```

## Agents

### 1. @ui-designer (DESIGN phase)

- **Model:** sonnet
- **Tools:** Read, Glob, Grep (read-only)
- **Role:** Produces UI/UX design specifications — NOT implementation code
- **Outputs:** design_rationale, wireframe, specifications, test_constraints
- **Must NOT:** Write implementation code, modify files, make backend decisions
- **Generic approach:** Reads CLAUDE.md for project's design system, styling conventions,
  component patterns. If no design system exists, follows platform best practices.

Based on cocomind's @ux-designer. Stripped of: CocoMind color palette, mascot system,
NativeWind/Reanimated specifics, dark mode rules.

### 2. @api-designer (DESIGN phase)

- **Model:** sonnet
- **Tools:** Read, Glob, Grep (read-only)
- **Role:** Designs API contracts — endpoints, schemas, error formats, auth requirements
- **Outputs:** Endpoint specifications with method, path, request/response schemas, status codes
- **Must NOT:** Write implementation code, modify files, make UI decisions
- **Generic approach:** Reads CLAUDE.md for project's API style (REST/GraphQL/gRPC),
  auth pattern, error format. Provides sensible REST defaults if unspecified.

Based on cocomind's @api-designer. Stripped of: Go hexagonal specifics, Supabase auth details,
CocoMind-specific endpoints, R2 signed URL patterns.

### 3. @infra-designer (DESIGN phase)

- **Model:** sonnet
- **Tools:** Read, Write, Edit, Glob, Grep, Bash
- **Role:** Infrastructure and DevOps specifications — IaC, CI/CD, containers, cloud resources
- **Outputs:** Infrastructure specs, pipeline configs, deployment plans
- **Must NOT:** Write application code, make product decisions
- **Generic approach:** Reads CLAUDE.md for project's cloud provider, IaC tool, CI/CD platform.
  Adapts to Terraform/OpenTofu/Pulumi/CloudFormation, GitHub Actions/GitLab CI/CircleCI, etc.
- **Human gate:** Infrastructure apply/deploy operations require explicit approval

Based on cocomind's @infrastructure-engineer + @devops-engineer merged. Stripped of:
OpenTofu/GCP/Cloudflare specifics, CocoMind FinOps rules, Supabase CLI patterns.

### 4. @tester (RED phase)

- **Model:** sonnet
- **Tools:** Read, Write, Edit, Glob, Grep, Bash
- **Role:** Writes FAILING tests before implementation exists (RED phase of TDD)
- **Iron rule:** Tests MUST FAIL when first run. If a test passes without new code, rewrite it.
- **Must NOT:** Write implementation code, modify existing implementation, make tests pass
- **Generic approach:** Reads CLAUDE.md for project's test framework, naming conventions,
  coverage requirements. Detects stack from project files (package.json, go.mod, Cargo.toml,
  etc.) and uses appropriate testing tools.
- **Coverage:** Happy path, edge cases, error scenarios, boundary values

Based on cocomind's @tester. Stripped of: testify/httptest/testcontainers-go specifics,
Jest/RNTL/Maestro specifics, CocoMind SOS/animation test patterns.

### 5. @implementer (GREEN + REFACTOR EXEC phases)

- **Model:** sonnet
- **Tools:** Read, Write, Edit, Glob, Grep, Bash
- **Role:** Writes MINIMUM code to make failing tests pass (GREEN), then applies approved
  refactoring plans (REFACTOR EXEC)
- **Iron rule:** Do not over-engineer. Do not add features not covered by tests.
- **Must NOT:** Write new tests, modify existing tests to make them pass, make architectural
  decisions, add functionality beyond what tests require
- **Generic approach:** Reads CLAUDE.md for project's architecture patterns, coding standards,
  forbidden patterns. Follows whatever conventions are defined there.

Based on cocomind's @implementer. Stripped of: Go hexagonal rules, React Native styling rules,
NativeWind/Reanimated specifics, SOS mandate, CocoMind-specific coding standards.

### 6. @architect (REFACTOR AUDIT phase)

- **Model:** opus
- **Tools:** Read, Glob, Grep (read-only)
- **Role:** Audits GREEN implementation against project constitution (CLAUDE.md), produces
  actionable refactoring plans
- **Outputs:** Structured Refactoring Plan with prioritized changes, risk levels, affected tests
- **Must NOT:** Write implementation code, write tests, approve own plan
- **Human gate:** MANDATORY. Owner must approve/modify/reject the refactoring plan before
  execution proceeds. This gate is never skipped.
- **Generic approach:** Reads CLAUDE.md to build audit checklist dynamically. Checks architecture
  boundaries, coding standards, security patterns, performance rules — whatever the project
  defines. If no CLAUDE.md exists, audits against industry best practices for the detected stack.

Based on cocomind's @architect. Stripped of: Go hexagonal checklist, React Native animation
performance checklist, CocoMind-specific cross-cutting checks.

### 7. @code-reviewer (QUALITY phase)

- **Model:** opus
- **Tools:** Read, Glob, Grep, Bash
- **Role:** Reviews all changes from the entire TDD cycle for correctness, readability,
  performance, and adherence to project conventions
- **Outputs:** Structured review report with strengths, must-fix issues, optional suggestions,
  metrics (tests, coverage, lint)
- **Must NOT:** Modify files, write code, approve PRs
- **Generic approach:** Reads CLAUDE.md for project's review standards. Runs project's linter
  and type checker. Checks coverage against project's threshold (defaults to 80% if unspecified).

Based on cocomind's @code-reviewer. Stripped of: golangci-lint/eslint specifics, CocoMind
architecture checklist items, bilingual (Spanish/English) check.

### 8. @security-auditor (QUALITY phase)

- **Model:** opus
- **Tools:** Read, Glob, Grep, Bash
- **Role:** Scans all changed files for security vulnerabilities, insecure patterns, and
  compliance gaps
- **Outputs:** Security audit report with severity-rated findings and remediation steps
- **Must NOT:** Modify files, write code, dismiss findings
- **Generic approach:** Applies OWASP Top 10 checks, scans for hardcoded secrets, checks
  auth/authz patterns, validates input sanitization. Reads CLAUDE.md for project-specific
  security requirements (e.g., RLS policies, signed URLs, encryption at rest).
- **Audit domains:** Authentication/authorization, data protection, API security, secret
  management, dependency vulnerabilities, forbidden code patterns

Based on cocomind's @security-auditor. Stripped of: Supabase RLS specifics, GDPR/CocoMind
data sensitivity rules, GCP infrastructure checks, CocoMind-specific forbidden patterns.

## Command: /triad

### Input

`$ARGUMENTS` — task description, issue reference, or Prompt Contract from specflow.

### Execution Flow (8 Phases)

```
Phase 1: ANALYSIS
  Parse task -> identify affected domains from project structure
  Determine which DESIGN agents are needed (if any)

Phase 2: DESIGN (optional, 1+ agents)
  If UI/UX work needed       -> @ui-designer
  If API contract work needed -> @api-designer
  If infra/DevOps work needed -> @infra-designer
  If pure logic (no new interfaces) -> skip DESIGN entirely
  Gate: Design specs are complete and actionable

Phase 3: RED (@tester)
  Write FAILING tests based on requirements + design specs
  Gate: Run tests, confirm they FAIL

Phase 4: GREEN (@implementer)
  Write MINIMUM code to make tests PASS
  Gate: Run tests, confirm ALL PASS

Phase 5: REFACTOR AUDIT (@architect)
  Audit GREEN code against CLAUDE.md constitution
  Produce Refactoring Plan
  CRITICAL HUMAN GATE: Owner approves/modifies/rejects plan
  DO NOT PROCEED without explicit approval

Phase 6: REFACTOR EXEC (@implementer)
  Apply ONLY approved changes from the plan
  Gate: Run tests, confirm ALL still PASS

Phase 7: QUALITY CHECKS (parallel)
  @code-reviewer  — code quality review
  @security-auditor — security scan
  Gate: No Critical or High severity issues
  If Critical/High found -> STOP, present to owner, fix before proceeding

Phase 8: COMPLETION
  Summary report: phases executed, files modified, test results, open items
```

### Domain Detection (Generic)

Instead of hardcoded directory checks (api/, mobile/), the orchestrator:

1. Reads project structure to identify domains (e.g., src/, lib/, packages/)
2. Reads CLAUDE.md for declared module boundaries
3. Infers from task description which domains are affected
4. Maps domains to DESIGN agents:
   - UI/frontend files -> @ui-designer
   - API/endpoint files -> @api-designer
   - Infrastructure/CI files -> @infra-designer
   - Other -> skip DESIGN

### Behavior Rules

1. **Strict sequencing** — Never advance past a phase until its gate passes
2. **Mandatory human gate** — REFACTOR AUDIT (Phase 5) always stops for owner approval.
   Quality check Critical/High findings also stop for owner.
3. **Role isolation** — Each agent stays in its lane:
   - Designers don't implement
   - Tester doesn't write feature code
   - Implementer doesn't rewrite tests
   - Architect doesn't write refactored code
   - Reviewer and auditor don't modify files
4. **Constitution as authority** — CLAUDE.md is loaded into every agent's context. It is
   the supreme authority for all conventions, patterns, and rules.
5. **No nesting** — Agents don't spawn subagents. Each completes work and returns to orchestrator.
6. **Cross-cutting tasks** — A single task touching multiple domains runs through ONE triad
   loop, not split into separate loops.

## Integration with specflow

```
/specflow:implement #49
  -> reads Prompt Contract from GitHub issue #49
  -> moves task to "In Progress (Triad Active)"
  -> checks: does /triad command exist?
     YES -> invokes /triad with Prompt Contract as $ARGUMENTS
     NO  -> falls back to superpowers:test-driven-development
  -> on REFACTOR gate: moves task to "HITL Review"
```

No cross-plugin skill references needed. specflow invokes /triad as a command.

## Completion Report Template

```markdown
## Triad Cycle Complete — [Feature/Task Name]

### Phases Executed
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

## What Is NOT In This Plugin

These are project-level concerns, not plugin concerns:

- Specific design systems (colors, typography, mascots)
- Specific architecture patterns (hexagonal, clean, layered)
- Specific test frameworks or naming conventions
- Specific CI/CD platforms or cloud providers
- Feature flag systems
- Observability/monitoring (too stack-specific for generic plugin)
- Content management workflows
- GDPR/compliance specifics

All of these are defined in the project's CLAUDE.md and read by agents at runtime.
