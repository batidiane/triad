# Triad — Multi-Agent TDD Orchestrator

A Claude Code plugin that orchestrates 8 specialized agents through a strict
Test-Driven Development cycle: **DESIGN > RED > GREEN > REFACTOR > QUALITY**.

Stack-agnostic — agents read your project's `CLAUDE.md` for conventions.

## Install

```bash
claude plugins add batidiane/triad
```

## Usage

```
/triad Implement user authentication with JWT tokens
/triad Fix the race condition in the cache invalidation logic
/triad Add pagination to the /api/users endpoint
```

The orchestrator parses your task, detects which domains are affected, and
dispatches specialized agents through 8 phases:

| Phase | Agent | What happens |
|-------|-------|-------------|
| 1. Analysis | orchestrator | Parse task, detect domains |
| 2. Design | @ui-designer / @api-designer / @infra-designer | Produce specs (optional) |
| 3. RED | @tester | Write failing tests |
| 4. GREEN | @implementer | Minimum code to pass tests |
| 5. Refactor Audit | @architect | Audit code, propose refactoring plan |
| 6. Refactor Apply | @implementer | Apply approved plan |
| 7. Quality | @code-reviewer + @security-auditor | Code review + security scan |
| 8. Completion | orchestrator | Summary report |

## Human Gates

The orchestrator stops and waits for your approval at:

- **Phase 5** — You review and approve/modify/reject the refactoring plan
- **Phase 7** — If Critical/High issues are found, you decide how to proceed

## CLAUDE.md Integration

Every agent reads your project's `CLAUDE.md` at the start of its task. Define your:

- Architecture patterns (hexagonal, clean, layered, etc.)
- Coding standards and forbidden patterns
- Test framework and naming conventions
- Design system tokens and styling rules
- Security requirements
- CI/CD and infrastructure conventions

Agents adapt to whatever you define. No `CLAUDE.md`? They fall back to industry
best practices for your detected tech stack.

## Works with specflow

If you also use [specflow](https://github.com/batidiane/specflow), the
`/specflow:implement` command automatically invokes `/triad` when available:

```
/specflow:implement #49
  -> reads Prompt Contract from GitHub issue
  -> invokes /triad with the contract
  -> manages task status transitions
```

## Agents

| Agent | Phase | Model | Access |
|-------|-------|-------|--------|
| @ui-designer | Design | Sonnet | Read-only |
| @api-designer | Design | Sonnet | Read-only |
| @infra-designer | Design | Sonnet | Read + Write + Bash |
| @tester | RED | Sonnet | Read + Write + Bash |
| @implementer | GREEN + Refactor | Sonnet | Read + Write + Bash |
| @architect | Refactor Audit | Opus | Read-only |
| @code-reviewer | Quality | Opus | Read + Bash |
| @security-auditor | Quality | Opus | Read + Bash |

## Customization

Override any agent at the project level by creating a file with the same name
in your `.claude/agents/` directory. Project-level agents take precedence over
plugin agents.

## License

MIT
