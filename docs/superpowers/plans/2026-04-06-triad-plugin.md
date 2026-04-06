# Triad Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a generic multi-agent TDD orchestrator plugin for Claude Code that works with any tech stack.

**Architecture:** A Claude Code plugin with 1 command (`/triad`), 8 agent definitions, and plugin metadata. All files are markdown/JSON — no application code. Agents are generic and read the project's CLAUDE.md at runtime for stack-specific conventions.

**Tech Stack:** Claude Code plugin system (markdown commands, markdown agents, JSON plugin config)

**Spec:** `docs/superpowers/specs/2026-04-06-triad-plugin-design.md`

---

## File Map

| File | Responsibility |
|------|---------------|
| `.claude-plugin/plugin.json` | Plugin identity, version, entry point |
| `.claude-plugin/marketplace.json` | GitHub marketplace discovery metadata |
| `package.json` | npm package metadata for plugin registry |
| `commands/triad.md` | Orchestrator command — 8-phase TDD flow with domain detection, gates, agent dispatch |
| `agents/ui-designer.md` | DESIGN phase — UI/UX specifications (read-only) |
| `agents/api-designer.md` | DESIGN phase — API contract specifications (read-only) |
| `agents/infra-designer.md` | DESIGN phase — Infrastructure + DevOps specs (read+write) |
| `agents/tester.md` | RED phase — writes failing tests only |
| `agents/implementer.md` | GREEN + REFACTOR — minimum code to pass tests |
| `agents/architect.md` | REFACTOR AUDIT — audits code, produces refactoring plans (read-only) |
| `agents/code-reviewer.md` | QUALITY phase — code review report (read-only) |
| `agents/security-auditor.md` | QUALITY phase — security audit report (read-only) |
| `LICENSE` | MIT license |
| `README.md` | Plugin documentation, install instructions, usage |

---

### Task 1: Plugin Scaffold

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`
- Create: `package.json`
- Create: `LICENSE`

- [ ] **Step 1: Create plugin.json**

Write to `.claude-plugin/plugin.json`:
```json
{
  "name": "triad",
  "version": "1.0.0",
  "description": "Multi-agent TDD orchestrator — DESIGN > RED > GREEN > REFACTOR > QUALITY",
  "author": "batidiane"
}
```

- [ ] **Step 2: Create marketplace.json**

Write to `.claude-plugin/marketplace.json`:
```json
{
  "name": "triad",
  "title": "Triad — Multi-Agent TDD Orchestrator",
  "description": "Orchestrates 8 specialized agents through a strict TDD cycle: DESIGN > RED > GREEN > REFACTOR > QUALITY. Stack-agnostic — reads your CLAUDE.md for project conventions.",
  "tags": ["tdd", "testing", "agents", "workflow", "orchestrator"],
  "source": {
    "type": "github",
    "url": "https://github.com/batidiane/triad"
  }
}
```

- [ ] **Step 3: Create package.json**

Write to `package.json`:
```json
{
  "name": "@batidiane/triad",
  "version": "1.0.0",
  "description": "Multi-agent TDD orchestrator plugin for Claude Code",
  "author": "batidiane",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/batidiane/triad.git"
  },
  "keywords": ["claude-code-plugin", "tdd", "agents", "orchestrator"]
}
```

- [ ] **Step 4: Create LICENSE**

Write MIT license file with `Copyright (c) 2026 batidiane` to `LICENSE`.

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/ package.json LICENSE
git commit -m "feat: plugin scaffold — plugin.json, marketplace.json, package.json, LICENSE"
```

---

### Task 2: @ui-designer Agent

**Files:**
- Create: `agents/ui-designer.md`

- [ ] **Step 1: Write the agent definition**

The agent must include these sections in the frontmatter and body:
- Frontmatter: name=ui-designer, description (mentions UI/UX design, component styling, accessibility, read-only), tools=Read/Glob/Grep, model=sonnet
- Project Conventions section: reads CLAUDE.md for design system, styling conventions. Falls back to platform best practices if none defined.
- Role section: produces design_rationale, wireframe, specifications, test_constraints — NOT implementation code
- Must NOT section: write backend/implementation code, modify files, make data flow decisions
- Design Principles: accessibility first, responsive, animation contracts (trigger, timing, reduced motion fallback, performance), design tokens over arbitrary values
- Process: read CLAUDE.md, understand context, produce 4 outputs, reference project tokens

Source material: `/Users/bat/Dev/cocomind/.claude/agents/ux-designer.md` — genericize by removing CocoMind color palette, mascot system, NativeWind/Reanimated specifics, dark mode rules.

Write to `agents/ui-designer.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/ui-designer.md
git commit -m "feat: add @ui-designer agent — DESIGN phase, UI/UX specifications"
```

---

### Task 3: @api-designer Agent

**Files:**
- Create: `agents/api-designer.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=api-designer, description (mentions API contracts, endpoint specs, schemas, read-only), tools=Read/Glob/Grep, model=sonnet
- Project Conventions section: reads CLAUDE.md for API style (REST/GraphQL/gRPC), auth pattern, error format. Defaults to RESTful JSON if unspecified.
- Role section: defines contracts between clients and API — endpoints, methods, schemas, status codes, error formats, auth, caching
- Must NOT section: write implementation code, client code, DB queries, modify files
- API Specification Format: structured template for each endpoint (METHOD /path, Purpose, Auth, Request, Response, Errors, Notes)
- Process: read CLAUDE.md, read requirements, design contract, identify caching, specify service interface methods

Source material: `/Users/bat/Dev/cocomind/.claude/agents/api-designer.md` — genericize by removing Go hexagonal specifics, Supabase auth, CocoMind endpoints, R2 patterns.

Write to `agents/api-designer.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/api-designer.md
git commit -m "feat: add @api-designer agent — DESIGN phase, API contracts"
```

---

### Task 4: @infra-designer Agent

**Files:**
- Create: `agents/infra-designer.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=infra-designer, description (mentions IaC, CI/CD, containers, cloud resources), tools=Read/Write/Edit/Glob/Grep/Bash, model=sonnet
- Project Conventions section: reads CLAUDE.md for cloud provider, IaC tool, CI/CD platform. Detects from project files if unspecified.
- Scope: IaC modules, CI/CD workflows, container configs, cloud provisioning, secret management, deployment automation
- Must NOT: write application code, make product decisions, store secrets in files, apply without validation
- Infrastructure Rules: plan before apply, environments mirror, remote state, no ClickOps, secrets via manager
- CI/CD Rules: conditional paths, cache deps, quality gates, multi-stage Docker
- Human Gate: apply/deploy always requires human approval
- Process: read CLAUDE.md, understand requirements, write configs, validate, present plan

Source material: `/Users/bat/Dev/cocomind/.claude/agents/infrastructure-engineer.md` + `devops-engineer.md` — merged and genericized by removing OpenTofu/GCP/Cloudflare specifics, FinOps rules, Supabase CLI.

Write to `agents/infra-designer.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/infra-designer.md
git commit -m "feat: add @infra-designer agent — DESIGN phase, infrastructure + DevOps"
```

---

### Task 5: @tester Agent

**Files:**
- Create: `agents/tester.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=tester, description (mentions RED phase, failing tests, stack detection), tools=Read/Write/Edit/Glob/Grep/Bash, model=sonnet
- Project Conventions section: reads CLAUDE.md for test framework, naming, directory structure, coverage. Auto-detects from project files (go.mod, package.json, Cargo.toml, pyproject.toml).
- The Iron Rule: tests MUST FAIL, written BEFORE implementation
- Must NOT: write implementation, modify implementation files, make tests pass
- Test Quality Checklist: happy path, edge cases, error scenarios, boundary values
- Test Structure: Arrange-Act-Assert, table-driven/parameterized for multiple scenarios
- Process: read requirements + design outputs, read existing code, write failing tests, run to confirm FAIL, report

Source material: `/Users/bat/Dev/cocomind/.claude/agents/tester.md` — genericize by removing testify/httptest/testcontainers specifics, Jest/RNTL/Maestro specifics, SOS/animation patterns.

Write to `agents/tester.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/tester.md
git commit -m "feat: add @tester agent — RED phase, failing tests"
```

---

### Task 6: @implementer Agent

**Files:**
- Create: `agents/implementer.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=implementer, description (mentions GREEN phase, minimum code, REFACTOR application), tools=Read/Write/Edit/Glob/Grep/Bash, model=sonnet
- Project Conventions section: reads CLAUDE.md for architecture patterns, coding standards, forbidden patterns
- The Iron Rule: minimum code, no over-engineering, no features beyond tests
- Must NOT: write tests, modify tests, add untested functionality, make architecture decisions, skip running tests
- GREEN Phase Process: read failing tests, read existing code, implement minimum, run tests after every change, iterate to 100% green
- REFACTOR Phase Process: only after approved plan, apply ONLY approved changes, don't alter behavior, don't break tests, run full suite

Source material: `/Users/bat/Dev/cocomind/.claude/agents/implementer.md` — genericize by removing Go hexagonal rules, React Native styling, NativeWind/Reanimated, SOS mandate.

Write to `agents/implementer.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/implementer.md
git commit -m "feat: add @implementer agent — GREEN + REFACTOR phases"
```

---

### Task 7: @architect Agent

**Files:**
- Create: `agents/architect.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=architect, description (mentions REFACTOR audit, constitution compliance, refactoring plans, read-only), tools=Read/Glob/Grep, model=opus
- Project Conventions section: reads CLAUDE.md as supreme authority, builds audit checklist dynamically. Falls back to best practices if no CLAUDE.md.
- Role: quality gate between GREEN and REFACTOR, produces actionable refactoring plans
- Must NOT: write implementation, write tests, make UI decisions, approve own plan, modify files
- Audit Process: read CLAUDE.md, read changed files, check architecture/coding/security/performance, produce plan, STOP for approval
- Refactoring Plan Format: Summary, Changes (file, issue, fix, risk, tests affected), No-Change Confirmations, Questions for Owner

Source material: `/Users/bat/Dev/cocomind/.claude/agents/architect.md` — genericize by removing Go hexagonal checklist, React Native animation checklist, CocoMind cross-cutting checks.

Write to `agents/architect.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/architect.md
git commit -m "feat: add @architect agent — REFACTOR AUDIT phase, refactoring plans"
```

---

### Task 8: @code-reviewer Agent

**Files:**
- Create: `agents/code-reviewer.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=code-reviewer, description (mentions code quality, correctness, adherence to conventions, read-only), tools=Read/Glob/Grep/Bash, model=opus
- Project Conventions section: reads CLAUDE.md for review standards, builds checklist dynamically
- Role: catches issues before owner reviews, produces structured report
- Must NOT: modify files, write code, approve changes, give vague feedback
- Review Process: read requirements, run tests, check coverage, run linters, read every changed file, apply checklist, produce report
- Review Checklist: Correctness, Architecture, Security, Performance, Testing, Style
- Review Report Format: Summary (APPROVE/REQUEST CHANGES), Strengths, Issues (Must Fix with file:line), Suggestions, Metrics

Source material: `/Users/bat/Dev/cocomind/.claude/agents/code-reviewer.md` — genericize by removing golangci-lint/eslint specifics, CocoMind architecture items, bilingual check.

Write to `agents/code-reviewer.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/code-reviewer.md
git commit -m "feat: add @code-reviewer agent — QUALITY phase, code review"
```

---

### Task 9: @security-auditor Agent

**Files:**
- Create: `agents/security-auditor.md`

- [ ] **Step 1: Write the agent definition**

The agent must include:
- Frontmatter: name=security-auditor, description (mentions security audits, OWASP, vulnerabilities, read-only), tools=Read/Glob/Grep/Bash, model=opus
- Project Conventions section: reads CLAUDE.md for project-specific security requirements, layers on top of generic OWASP checks
- Must NOT: modify files, write code, dismiss findings, assume patterns are safe
- Audit Domains: Authentication/Authorization, Input Validation/Injection, Secret Management, Data Protection, API Security, Dependency Security, Forbidden Patterns
- Audit Report Format: Risk Summary table (severity/count/status), Findings (severity-coded with location/description/impact/remediation/references), Passed Checks, Recommendations
- Process: read CLAUDE.md, read changed files, run vulnerability scanners, scan for secrets/forbidden patterns, apply full checklist, produce report

Source material: `/Users/bat/Dev/cocomind/.claude/agents/security-auditor.md` — genericize by removing Supabase RLS, GDPR/CocoMind data rules, GCP infrastructure checks.

Write to `agents/security-auditor.md`.

- [ ] **Step 2: Commit**

```bash
git add agents/security-auditor.md
git commit -m "feat: add @security-auditor agent — QUALITY phase, security audit"
```

---

### Task 10: /triad Orchestrator Command

**Files:**
- Create: `commands/triad.md`

This is the most important file — the orchestrator that ties everything together.

- [ ] **Step 1: Write the orchestrator command**

The command must include:
- Frontmatter: description (mentions TDD orchestration, 8 phases, stack-agnostic), argument-hint="Task description, issue reference, or Prompt Contract", allowed-tools=Read/Write/Edit/Bash/Glob/Grep/Agent
- User Input section: `$ARGUMENTS` with instruction to consider it
- Available Agents table: all 8 agents with role, model, tools, phase
- Execution Flow diagram: 8 phases in ASCII art
- Detailed Steps (1-8):
  - Step 1 ANALYSIS: parse task, read CLAUDE.md, detect domains, map to design agents. Domain detection table (UI->ui-designer, API->api-designer, infra->infra-designer, pure logic->skip design)
  - Step 2 DESIGN: conditional dispatch to design agents with handoff contents and gate criteria
  - Step 3 RED: delegate to @tester with design outputs + requirements. Gate: tests FAIL
  - Step 4 GREEN: delegate to @implementer with failing tests + design specs. Gate: tests PASS
  - Step 5 REFACTOR AUDIT: delegate to @architect. CRITICAL HUMAN GATE — stop, present plan, wait for approval
  - Step 6 REFACTOR APPLY: delegate to @implementer with approved plan ONLY. Gate: tests still PASS
  - Step 7 QUALITY CHECKS: dispatch @code-reviewer + @security-auditor in PARALLEL. Gate: no Critical/High. Stop if found.
  - Step 8 COMPLETION: summary report using the template from the spec
- Behavior Rules: strict sequencing, mandatory human gate, role isolation (list all 8 agents with their constraint), constitution as authority, no nesting, cross-cutting tasks stay in one loop

Source material: `/Users/bat/Dev/cocomind/.claude/commands/triad.md` — genericize by removing CocoMind module references (api/, mobile/), Go/React Native test commands, PostHog feature flags, observability agent, content-manager agent.

Write to `commands/triad.md`.

- [ ] **Step 2: Commit**

```bash
git add commands/triad.md
git commit -m "feat: add /triad orchestrator command — 8-phase TDD with agent dispatch"
```

---

### Task 11: README

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Include:
- Title: "Triad — Multi-Agent TDD Orchestrator"
- One-line description: stack-agnostic, reads CLAUDE.md
- Install: `claude plugins add batidiane/triad`
- Usage: 3 example invocations showing different task types
- Phase table: all 8 phases with agent and description
- Human Gates section: Phase 5 refactor approval, Phase 7 critical/high findings
- CLAUDE.md Integration section: what to define (architecture, standards, tests, design system, security, CI/CD)
- Works with specflow section: how /specflow:implement invokes /triad
- Agents table: all 8 with phase, model, access level
- Customization section: override agents via project .claude/agents/
- License: MIT

Write to `README.md`.

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with install, usage, and agent reference"
```

---

### Task 12: Final Push

- [ ] **Step 1: Push all commits to remote**

```bash
git push origin main
```

- [ ] **Step 2: Verify on GitHub**

```bash
gh repo view batidiane/triad --web
```
