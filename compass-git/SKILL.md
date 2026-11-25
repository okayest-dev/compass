---
name: compass-git
description: ""Generic git workflow skills: atomic commit splitting, stacked PR rebase, bisect regression, conflict resolution, safe history rewrite, undo recovery, branch cleanup. Tracker-agnostic, works with any git repo.""
---

# Compass Git

Generic git workflow skills for spec-driven development. These are not tied to any tracker or platform — they work with any git repository.

## Consumes / produces / hands off

- **Consumes:** a git repository with work in progress (uncommitted changes, stacked branches, merge conflicts, lost commits, regression).
- **Produces:** clean history (atomic commits, rebased stacks, resolved conflicts, recovered work, identified regression commit).
- **Called from:** `compass-implement` (before commit), `compass-quick-implement` (before commit), any skill needing clean git history, or directly by the user.
- **Hands off to:** the calling skill continues with clean history.

## Skills

Each sub-skill is a standalone operation. Invoke by name: `compass-git atomic-commit`, `compass-git stacked-rebase`, etc.

### 1. Atomic Commit (`atomic-commit`)

Split working-tree changes into single-purpose commits, ordered so each builds and is messaged to match the repository's style.

**Process:**

1. Run `git status` and `git diff` to see all changes.
2. Group changes by logical purpose (one behaviour, one fix, one refactor per group). A group may span multiple files.
3. For each group, create a commit:
   - Stage only the hunks for that group (`git add -p`).
   - Verify the staged change builds and tests pass.
   - Write a commit message following the repository's convention (check recent commits for pattern: conventional commits, Jira keys, Linear IDs, etc.).
   - Commit.
4. Order commits so each builds on the previous (dependencies first).
5. Run the full test suite once at the end.

**Output:** a series of commits, each passing build and tests, with consistent messages.

### 2. Stacked PR Rebase (`stacked-rebase`)

Restack dependent PRs after a lower one is squash-merged or amended.

**Prerequisites:** git 2.38+ (for `git rebase --update-refs`), optionally `gh` CLI to read and retarget PR bases.

**Process:**

1. Identify the stack: branches that depend on each other (check `git log --oneline --graph` or `gh pr list --json baseRefName,headRefName`).
2. Find the branch that was merged/amended (the "base shift").
3. For each branch above it in the stack, in order from bottom to top:
   - `git checkout <branch>`
   - `git rebase --update-refs <new-base>` (where `<new-base>` is the updated branch or commit).
   - Resolve any conflicts (see `conflict-resolution` below).
   - Force-push if the branch was already pushed: `git push --force-with-lease`.
   - If using `gh`, retarget the PR: `gh pr edit --base <new-base>`.
4. Verify the whole stack builds and tests pass.

**Output:** restacked branches with correct bases, PRs retargeted if applicable.

### 3. Conflict Resolution (`conflict-resolution`)

Resolve conflicts by combining intent, not picking a side.

**Process:**

1. For each conflicted file, read both sides (`git diff --ours`, `git diff --theirs`).
2. Understand the intent of each change (what behaviour each side adds/changes).
3. Produce a merged version that preserves both intents. Do not simply pick one side or the other.
4. If intents are genuinely incompatible, escalate to the decision owner (hand off to `compass-navigate`).
5. Stage the resolved file (`git add <file>`).
6. Continue the operation (rebase, merge, cherry-pick).

**Output:** resolved conflicts with combined intent, no `<<<<<<<` markers.

### 4. Safe History Rewrite (`safe-rewrite`)

Decide revert vs. rewrite and push with a lease.

**Process:**

1. Determine if the change is already public (pushed to a shared remote, or a PR exists).
2. If **not public**: rewrite freely (amend, rebase, squash, drop commits).
3. If **public**:
   - Prefer `git revert` for a clean, traceable undo.
   - Rewrite only if the history is actively harmful (exposed secrets, large binaries, PII) and all collaborators can coordinate a force-push.
   - If rewriting: `git push --force-with-lease` (not `--force`) to avoid clobbering concurrent pushes.
   - Notify all collaborators before and after.

**Output:** clean history with safe push semantics.

### 5. Undo Recovery (`undo-recovery`)

Get lost commits, branches, and stashes back via the reflog.

**Process:**

1. Run `git reflog` (or `git reflog <branch>` for a specific branch).
2. Identify the lost commit by message, hash, or time.
3. Recover:
   - Commit: `git checkout <hash>` then `git checkout -b <recovery-branch>`.
   - Branch: `git checkout -b <branch> <hash>`.
   - Stash: `git stash list`, then `git stash apply stash@{N}`.
4. Verify the recovered work builds and tests pass.

**Output:** recovered work on a new branch or applied stash.

### 6. Bisect Regression (`bisect-regression`)

Find the commit that broke something.

**Process:**

1. Identify a known-good commit (`git bisect start <bad> <good>`).
2. Provide a test script that exits 0 for good, non-zero for bad (e.g., `./run-tests.sh` or a specific test).
3. Run `git bisect run <test-script>`.
4. The reported commit is the first bad one. Examine it (`git show <commit>`).
5. `git bisect reset` to return to the original branch.

**Output:** the first bad commit, with context.

### 7. Branch Cleanup (`branch-cleanup`)

Delete merged branches, including squash-merged ones, without losing work.

**Process:**

1. List all branches: `git branch -a`.
2. For each local branch:
   - Check if it is merged into the base branch (`git merge-base --is-ancestor <branch> <base>`).
   - For squash-merged branches (no merge commit): check if the branch's tip commit is an ancestor of base's history, or compare the diff (`git diff <branch> <base> --stat` — empty means squash-merged).
   - If merged and no unmerged work: `git branch -d <branch>` (or `-D` if forced).
3. For remote-tracking branches: `git fetch -p` to prune deleted remotes, then delete local counterparts if merged.
4. Never delete a branch with unmerged work or an open PR.

**Output:** cleaned branch list, no lost work.

## Repository conventions detection

Before any operation, detect the repository's conventions by inspecting recent history (last 20 commits):

- **Commit message format**: conventional commits (`feat:`, `fix:`, `chore:`), Jira keys (`PROJ-123`), Linear IDs (`ENG-456`), or custom.
- **Branch naming**: `feature/`, `fix/`, `chore/`, ticket-based (`PROJ-123-description`), stacked (`stack/base`, `stack/feature`).
- **Base branch**: `main`, `master`, `develop`, `trunk`.
- **Push policy**: force-with-lease required, protected branches.

Apply detected conventions in `atomic-commit` (message format), `stacked-rebase` (branch naming), `safe-rewrite` (push policy).

## Notes

- These skills are **generic** — they do not assume GitHub, GitLab, Jira, Linear, or any specific tracker. They work with bare git.
- `stacked-rebase` optionally uses `gh` CLI if available; if not, it works with git alone (manual PR retargeting).
- `bisect-regression` requires a test script; the calling skill or user must provide one.
- All skills verify build and tests pass before considering an operation complete.