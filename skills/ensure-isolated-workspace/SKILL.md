---
name: ensure-isolated-workspace
description: Use before implementation when workspace isolation may be needed - reuses an existing isolated workspace, allows clean in-place work for simple tasks, or creates a worktree when isolation is required
---

# Ensure Isolated Workspace

Choose the safest practical workspace setup before implementation begins.

**Core principle:** Reuse isolation when it already exists. Allow clean in-place work when it is safe. Create a worktree only when isolation is actually needed.

**Announce at start:** "I'm using the ensure-isolated-workspace skill to prepare a safe workspace."

## Valid Outcomes

This skill has exactly three valid outcomes:

1. Reuse an existing linked worktree
2. Allow clean in-place execution
3. Create a new worktree

Do not invent a fourth mode.

## Step 1: Detect the Current Environment

Run:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
git status --short
```

Interpretation:

- If `GIT_DIR` differs from `GIT_COMMON`, you are already inside a linked worktree. Reuse it.
- If `GIT_DIR` equals `GIT_COMMON`, you are in the main checkout or a normal checkout.
- If `git status --short` is empty, the workspace is clean.
- If `BRANCH` is `main` or `master`, treat that as a stronger signal that isolation may be needed.

## Step 2: Evaluate the Need for Isolation

Check these signals before choosing a workspace mode:

- Is the current directory already a linked worktree?
- Is the working tree clean?
- Is the current branch `main` or `master`?
- Is the task large, risky, or multi-step?
- Will parallel agents or subagents work on the task?
- Did the user explicitly request a worktree?
- Did the user explicitly say not to use a worktree?

## Step 3: Choose the Workspace Mode

### Outcome A: Reuse Existing Linked Worktree

If you are already inside a linked worktree:

1. Reuse it
2. Run project setup
3. Run baseline verification
4. Report that the existing isolated workspace is being reused

Do NOT create another nested worktree.

### Outcome B: Allow Clean In-Place Execution

Allow in-place work only when all of these are true:

- not already in a linked worktree
- working tree is clean
- current branch is not `main` or `master`, unless the user explicitly accepts that risk
- task is small enough that extra isolation would be overhead
- no parallel or subagent execution requires separate workspace state

If the user explicitly says not to use a worktree, you may choose in-place execution even when you would otherwise prefer isolation, but you must state the tradeoff clearly.

Report with language like:

```text
Workspace is clean and task scope is small. Proceeding in-place without creating a worktree.
```

### Outcome C: Create a New Worktree

Create a worktree when isolation is needed, including these cases:

- current branch is `main` or `master`
- working tree is dirty and risks cross-task contamination
- task is large, risky, or long-running
- parallel or subagent execution is about to start
- the user explicitly requests isolated work

When worktree creation is needed, use the directory selection and safety rules below.

## Step 4: Directory Selection for New Worktrees

Follow this priority order:

### 1. Check Existing Directories

```bash
ls -d .worktrees 2>/dev/null
ls -d worktrees 2>/dev/null
```

If both exist, `.worktrees` wins.

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

If preference is specified, use it without asking.

### 3. Ask User

If no directory exists and no preference is documented:

```text
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Step 5: Safety Verification for New Worktrees

### For Project-Local Directories

Before creating a project-local worktree:

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

If the chosen directory is not ignored:

1. Add the appropriate ignore rule
2. Commit the fix
3. Continue

### For Global Directories

No repository `.gitignore` verification is needed.

## Step 6: Create the Worktree When Required

```bash
project=$(basename "$(git rev-parse --show-toplevel)")

case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

## Step 7: Run Project Setup

Auto-detect and run project setup:

```bash
if [ -f package.json ]; then npm install; fi
if [ -f Cargo.toml ]; then cargo build; fi
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi
if [ -f go.mod ]; then go mod download; fi
```

## Step 8: Verify Clean Baseline

Run the project-appropriate baseline verification:

```bash
npm test
cargo test
pytest
go test ./...
```

If baseline fails:

- report the failures
- ask whether to proceed or investigate first

Do not silently continue.

## Step 9: Report the Chosen Mode

Always report which mode was chosen:

- existing linked worktree reused
- clean in-place execution approved
- new worktree created

Examples:

```text
Already in an isolated workspace at <path> on branch <name>. Baseline verified. Ready to implement.
```

```text
Workspace is clean and task scope is small. Proceeding in-place on branch <name>. Baseline verified. Ready to implement.
```

```text
Created isolated workspace at <path>. Baseline verified. Ready to implement.
```

## Red Flags

**Never:**

- create a nested worktree when already inside one
- proceed in-place on a dirty workspace without explicit user approval
- proceed in-place on `main` or `master` by accident
- skip setup or baseline verification
- remove the user's choice if they explicitly reject a worktree

**Always:**

- decide workspace mode explicitly before implementation
- reuse isolation when it already exists
- keep in-place execution limited to safe cases
- create a worktree when risk or coordination needs justify it

## Integration

**Called by:**

- **brainstorming** - REQUIRED before implementation begins
- **subagent-driven-development** - REQUIRED before executing tasks
- **executing-plans** - REQUIRED before executing tasks
- Any skill needing a safe implementation workspace

**Pairs with:**

- **finishing-a-development-branch** - Completes work regardless of whether execution happened in-place or in a worktree
