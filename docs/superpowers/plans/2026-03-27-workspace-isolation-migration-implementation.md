# Workspace Isolation Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Introduce `ensure-isolated-workspace` as the new primary workspace setup skill, keep `using-git-worktrees` as a compatibility shim, and update the surrounding workflow and docs to stop treating new worktree creation as mandatory.

**Architecture:** Add a new top-level skill that chooses among reusing an existing linked worktree, allowing clean in-place execution, or creating a worktree only when isolation is needed. Keep the old worktree skill as a deprecated compatibility entrypoint that forwards users to the new policy. Update dependent skills and Codex docs so the workflow consistently talks about ensuring a safe workspace rather than always creating one.

**Tech Stack:** Markdown skill files, repository documentation, Git workflow guidance

**Spec:** `docs/superpowers/specs/2026-03-27-workspace-isolation-migration-design.md`

---

## File Structure

| File | Responsibility | Action |
|---|---|---|
| `skills/ensure-isolated-workspace/SKILL.md` | New primary workspace-isolation workflow | Create |
| `skills/using-git-worktrees/SKILL.md` | Deprecated compatibility skill | Rewrite |
| `skills/writing-plans/SKILL.md` | Plan context wording | Update |
| `skills/executing-plans/SKILL.md` | Integration dependency wording | Update |
| `skills/subagent-driven-development/SKILL.md` | Integration dependency wording | Update |
| `skills/finishing-a-development-branch/SKILL.md` | Cleanup semantics when no worktree exists | Update |
| `docs/README.codex.md` | User-facing Codex workflow docs | Update |

### Task 1: Add `ensure-isolated-workspace`

**Files:**
- Create: `skills/ensure-isolated-workspace/SKILL.md`

- [ ] **Step 1: Write the new skill frontmatter and overview**

Create a new skill with:

```markdown
---
name: ensure-isolated-workspace
description: Use before implementation when workspace isolation may be needed - reuses an existing isolated workspace, allows clean in-place work for simple tasks, or creates a worktree when isolation is required
---
```

The body must define the three valid outcomes:
- reuse an existing linked worktree
- allow clean in-place execution
- create a worktree when isolation is required

- [ ] **Step 2: Define the decision process**

Document a concrete evaluation order:
- detect whether current directory is already a linked worktree
- inspect whether the worktree is clean
- inspect whether branch is `main` or `master`
- evaluate whether the task is large, risky, or parallelized
- respect explicit user instruction to avoid or require worktrees

- [ ] **Step 3: Preserve the useful setup and baseline checks**

Include setup/test guidance from the old worktree flow:
- auto-detect project setup
- run baseline verification
- stop and ask if baseline fails
- report the chosen workspace mode clearly

- [ ] **Step 4: Explain the policy**

Write explicit policy text:
- small, clean tasks may run in-place
- unsafe contexts require isolation
- existing linked worktrees should be reused
- worktrees remain a supported tool, but no longer the only valid path

### Task 2: Convert `using-git-worktrees` into a compatibility shim

**Files:**
- Modify: `skills/using-git-worktrees/SKILL.md`

- [ ] **Step 1: Rewrite the description and overview**

Mark the skill as deprecated for new workflows. State that new workflows should use `ensure-isolated-workspace`.

- [ ] **Step 2: Keep it functional as a compatibility entrypoint**

Make the skill instruct the agent to:
- announce that this is a compatibility path
- immediately invoke `ensure-isolated-workspace`
- prefer worktree creation if isolation is needed and the user has not explicitly said otherwise

- [ ] **Step 3: Remove the old detailed implementation body**

Do not leave two conflicting workflow definitions in the same file. Replace the old procedural worktree-creation content with a short compatibility policy and migration note.

### Task 3: Update dependent skills

**Files:**
- Modify: `skills/writing-plans/SKILL.md`
- Modify: `skills/executing-plans/SKILL.md`
- Modify: `skills/subagent-driven-development/SKILL.md`
- Modify: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: Update `writing-plans` context**

Replace wording that assumes a dedicated worktree already exists with wording that assumes either:
- an existing isolated workspace, or
- a clean approved in-place workspace

- [ ] **Step 2: Update integration references**

In `executing-plans` and `subagent-driven-development`, replace required references to `using-git-worktrees` with `ensure-isolated-workspace`.

- [ ] **Step 3: Update finishing behavior**

In `finishing-a-development-branch`, remove the assumption that cleanup always means removing a worktree created by the startup skill. Clarify:
- cleanup only applies when a removable worktree actually exists
- in-place execution has no worktree cleanup step
- externally managed worktrees should not be removed by default

### Task 4: Update Codex docs

**Files:**
- Modify: `docs/README.codex.md`

- [ ] **Step 1: Explain the new primary skill**

Add a short explanation that Codex workflow setup may now:
- reuse an existing isolated workspace
- allow clean in-place execution for small tasks
- create a worktree only when isolation is needed

- [ ] **Step 2: Keep compatibility mention concise**

Mention that `using-git-worktrees` remains for compatibility with older prompts and docs, but `ensure-isolated-workspace` is the preferred workflow concept.

### Task 5: Verify consistency

**Files:**
- Verify only

- [ ] **Step 1: Search for stale mandatory-creation language**

Run targeted searches for:
- `using-git-worktrees`
- `dedicated worktree`
- `Set up isolated workspace before starting`
- `created by brainstorming skill`
- `Cleans up worktree created by that skill`

Expected result: any remaining references should be intentional compatibility notes, not the primary workflow description.

- [ ] **Step 2: Check repository status**

Run:

```bash
git status --short
```

Expected result: only the intended skill/doc files for this migration are modified or added.
