# Workspace Isolation Migration Design

**Date:** 2026-03-27

**Goal:** Replace the hard requirement to always create a git worktree with a clearer workspace-isolation workflow that can reuse existing isolation, allow clean in-place work when appropriate, and still create a worktree when needed.

## Context

The current `superpowers` workflow strongly couples implementation to `using-git-worktrees`.

That creates friction in environments where isolation is already provided externally, such as Codex-managed worktrees. In those cases, forcing another worktree is redundant and can make path handling, cleanup, and user expectations more confusing.

There is also a second concern: some repositories have tests or build steps that are sensitive to absolute paths, shared runtime directories, or generated intermediate files. A rigid "always create a new worktree" rule can surface those weaknesses and make the workflow feel more fragile than necessary.

## Conclusions From Analysis

### 1. Worktrees are useful, but not a hard prerequisite

`worktree` is an engineering isolation tool, not a prerequisite for `superpowers` to function.

If the hard requirement is removed, the following skills still remain valid:

- `brainstorming`
- `writing-plans`
- `test-driven-development`
- `systematic-debugging`
- `requesting-code-review`
- `verification-before-completion`
- `finishing-a-development-branch` (with adjusted cleanup behavior)

What changes is not whether `superpowers` works, but how strongly it enforces isolation.

### 2. The real requirement is isolation, not necessarily worktree creation

The valuable parts of the current workflow are:

- avoiding cross-task contamination
- starting from a clean baseline
- making setup and verification explicit
- supporting parallel or multi-agent work safely

Those goals do not require unconditional `git worktree add`.

### 3. Current behavior is too specific for mixed environments

In repositories or host environments where:

- the host already launches work in an isolated worktree
- the user only wants a small single-task change
- tests or generated files are sensitive to path changes

forcing another worktree adds complexity without proportional benefit.

### 4. The current naming is misleading for the desired future behavior

If the desired behavior is:

- reuse an existing linked worktree
- optionally work in-place when the workspace is clean and the task is simple
- create a new worktree only when isolation is actually needed

then `using-git-worktrees` is no longer the clearest primary interface.

The clearer primary concept is: `ensure-isolated-workspace`.

## Recommended Direction

Adopt option 2 from the analysis:

- add a new primary skill: `ensure-isolated-workspace`
- do not immediately delete `using-git-worktrees`
- keep `using-git-worktrees` temporarily as a compatibility shim

This gives the project a cleaner conceptual model without breaking existing references all at once.

## Target Behavior

`ensure-isolated-workspace` should implement this policy:

1. If already inside a linked worktree, reuse it.
2. If not in a linked worktree, but the workspace is clean and the task is small, allow in-place execution.
3. If isolation is needed, create a worktree.
4. If the user explicitly does not want a worktree, allow in-place execution but make the tradeoff explicit.

## Decision Rules

The skill should evaluate at least these signals:

- whether the current directory is already a linked worktree
- whether the working tree is clean
- whether the current branch is `main` or `master`
- whether the task is large, risky, or multi-step
- whether parallel or subagent work is expected
- whether the user explicitly requested or rejected worktree usage

## Suggested Policy

Default policy:

- do not require a new worktree for every task
- do require a safe workspace decision before implementation begins

Mandatory isolation triggers:

- current branch is `main` or `master`
- working tree is dirty in a way that risks cross-task contamination
- multi-agent or parallel execution is about to start
- the task is substantial enough that rollback and cleanup matter
- the user explicitly requests isolation

Allowed in-place execution:

- current workspace is already isolated, or
- current workspace is clean, on an acceptable branch, and the task is small enough that additional isolation would be overhead

## Compatibility Recommendation

Do not delete `using-git-worktrees` yet.

Instead:

- mark it as deprecated for new workflows
- point new integrations to `ensure-isolated-workspace`
- optionally keep `using-git-worktrees` as a thin compatibility wrapper that delegates to the new policy

This avoids breaking:

- existing skill references
- old plans and specs
- user prompts that mention `using-git-worktrees`

## Files Expected To Change In The Next Step

The next migration plan should cover at least:

- `skills/ensure-isolated-workspace/SKILL.md` (new)
- `skills/using-git-worktrees/SKILL.md` (deprecate or wrap)
- `skills/writing-plans/SKILL.md`
- `skills/executing-plans/SKILL.md`
- `skills/subagent-driven-development/SKILL.md`
- `skills/finishing-a-development-branch/SKILL.md`
- any documentation that still describes worktree creation as mandatory

## Next Step To Produce

The next artifact should be a concrete migration plan that defines:

- the exact `ensure-isolated-workspace` skill contract
- whether `using-git-worktrees` becomes a wrapper or a deprecated fallback
- the exact edits required in dependent skills
- how finishing/cleanup behaves when no worktree was created
- how to describe the new behavior in Codex-facing documentation

## Non-Goals

This design note does not yet define:

- the final wording of each skill file
- exact migration commit boundaries
- test cases for the documentation changes
- whether the old skill is removed in a later release

Those belong in the follow-up migration plan.
