---
name: ralph
description: Issue planning and atomic commit methodology. Plan work, implement with independently-valid atomic commits, and enforce pre-commit quality gates. Use when user says "ralph" or needs structured implementation with disciplined commits.
---

# Ralph

Issue planning and atomic commit methodology. Plan work, then implement with disciplined, independently-valid commits and mandatory quality gates.

## Quick Start

```
User: "Implement #2"
1. Fetch issues from tracker
2. Prepare plan with ordered atomic steps
3. Implement each step as one atomic commit
4. Run pre-commit gate before every commit
```

## Workflow

### 1. Fetch Issues

Read `docs/agents/issue-tracker.md` to determine the tracker CLI (`glab`, `gh`, etc.). For each issue reference (number, URL, or path), fetch the full body and comments. Parse acceptance criteria, blockers, and context.

### 2. Prepare Plan

For each issue, write a plan covering:

- **Goal**: What "done" looks like (from acceptance criteria)
- **Files**: Which files will be created/modified
- **Steps**: Ordered implementation steps, each a potential atomic commit
- **Tests**: What to test and how

Save plan to `.sisyphus/plans/issue-<N>.md`.

### 3. Implement

Work through the plan step by step:

1. Start with `todowrite` to track progress
2. **Each step = one atomic commit** with a clear, self-contained message
3. Before each commit, run the pre-commit gate (see below)
4. Fix all diagnostics/lint/type issues **before** staging the commit

#### Atomic Commit Rules

- **One logical change per commit**: do not bundle unrelated refactors with feature work
- **Each commit must be independently valid**: if you stop after any commit, the codebase should be in a working state (compiles, tests pass)
- **Commit message format**:
  ```
  type: short description

  Details about what changed and why. Reference issue number.
  ```
- **No `as any`, `@ts-ignore`, or `@ts-expect-error`** in commits
- If a step breaks something, fix it in the same commit — do not leave broken state for a follow-up

#### Pre-Commit Gate (MANDATORY)

Before every commit, ALL of these must pass:

1. `lsp_diagnostics` — zero errors on changed files
2. Linter — zero new warnings/errors
3. Type checker — clean run
4. Tests — full test suite passes (or a targeted subset if full suite is slow, but document why)

If any check fails, fix the root cause. Do not skip, suppress, or defer failures.

### 4. Test

Run the full project test suite:

```bash
npm test / pytest / cargo test / etc.
```

- All tests must pass. Do NOT delete failing tests.
- If no tests exist and the implementation warrants them, write tests first before considering the task done.
- If a test fails due to your changes, fix the implementation — not the test.

## Additional Rules

- **Issue linking**: Always reference the issue number in commit messages (`refs #N`, `closes #N`).
- **Blocked issues**: If an issue lists blockers, verify blocking issues are resolved first.
- **Pre-commit gate is mandatory**: No exceptions for skipping diagnostics, lint, types, or tests.
