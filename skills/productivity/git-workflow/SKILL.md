---
name: git-workflow
description: End-to-end GitHub issue workflow using isolated git worktrees. Takes an issue, delegates implementation to a subagent with atomic commits, creates a PR, and handles the full review-to-merge cycle. Use when user says "implement issue", "work on issue", "create PR for issue", or references a GitHub issue number/URL.
---

# Git Workflow

End-to-end GitHub issue workflow with subagent-based implementation delegation.

## Quick Start

```
User: "Implement #42"
1. Fetch issue from GitHub
2. Determine active branch in repo root
3. Create worktree + branch
4. Delegate implementation to subagent
5. Verify subagent work
6. Push + create PR (auto-assign)
7. Wait for review comments
8. Address comments → push → reply
9. Confirm merge → cleanup
```

## Workflows

### 1. Setup

Read `docs/agents/` for project conventions (test commands, lint commands, AGENTS.md).

Identify **active branch** in repo root (`git branch --show-current`). This is the base branch the worktree and PR target.

### 2. Worktree + Branch

```bash
git worktree add .worktree/<branch-name> <active-branch>
cd .worktree/<branch-name>
git checkout -b <branch-name>
```

Branch naming: `{type}/{issue-number}-{kebab-description}`
Type: `feat`, `fix`, `chore`, `refactor`, `test`, `docs` (derive from issue labels/content).

### 3. Delegate Implementation

Do NOT implement yourself. Delegate the implementation to a `deep` or `unspecified-high` subagent (`task(category="deep" / "unspecified-high", load_skills=[...], run_in_background=true)`). The subagent prompt MUST include:

**TASK:** Implement the changes described in the issue plan at `.sisyphus/plans/issue-<N>.md`.
**EXPECTED OUTCOME:** All acceptance criteria met. Code implemented with atomic commits. Pre-commit gate passes.
**REQUIRED TOOLS:** `read`, `write`, `edit`, `glob`, `grep`, `bash` (for git), `lsp_diagnostics`.
**MUST DO:**
- Read the plan first, then implement step by step
- Each step = one atomic commit (see rules below)
- Run pre-commit gate before every commit: `lsp_diagnostics` + lint + type check — zero errors
- Use conventional commit messages: `type: short description`
- Each commit must be independently valid (compiles, tests pass)
- No `as any`, `@ts-ignore`, or `@ts-expect-error`
- Push changes: `git push origin <branch-name>`
**MUST NOT DO:**
- Do not create worktrees or branches (already done)
- Do not create PRs (handled in step 4)
- Do not modify code outside the implementation scope
- Do not force push
**CONTEXT:** Worktree is at `.worktree/<branch-name>`, branch is `<branch-name>`, base is `<active-branch>`.

### 4. Verify

After subagent returns, confirm:
- All commits are atomic and independently valid
- Diagnostics/lint/type checks pass
- Tests pass
- No suppressed type errors

### 5. Push + PR

```bash
git push origin <branch-name>
gh pr create \
  --base <active-branch> \
  --title "<type>: <issue-title>" \
  --body "$(gh issue view <number> --json body --jq .body)" \
  --assignee @me
```

PR body includes full issue description. Auto-assigns the invoker.

### 6. Review Loop

Wait for user signal (e.g. "review comments ready" or "address review").

1. Fetch PR review comments: `gh api /repos/:owner/:repo/pulls/:number/comments`
2. Fix all comments locally (batch fixes; delegate to subagent if significant)
3. Run tests + lint — must pass
4. Push: `git push origin <branch-name>` (no force push)
5. Reply to each comment with fix reference:
   ```
   gh api /repos/:owner/:repo/pulls/:number/comments/<comment-id>/replies \
     --field body="Fixed in <commit-hash>"
   ```

If all resolved, inform user the PR is ready for merge.

### 7. Merge + Cleanup

Wait for user to confirm PR is merged. Then:

```bash
# From repo root
git worktree remove .worktree/<branch-name>
git branch -d <branch-name>
```

---

## Atomic Commit Rules (for subagent prompts)

When delegating implementation, include these in the subagent prompt:

- **One logical change per commit**: no bundling unrelated refactors with feature work
- **Each commit must be independently valid**: codebase compiles and tests pass after every commit
- **Commit message format**:
  ```
  type: short description

  Details about what changed and why. Reference issue number.
  ```
- **No `as any`, `@ts-ignore`, or `@ts-expect-error`** in commits
- If a step breaks something, fix it in the same commit
