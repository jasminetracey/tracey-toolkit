---
name: resolve-protected-branch-conflicts
description: Resolve merge conflicts on a protected branch by merging into a detached repo state and pushing a new "-with-conflicts-resolved" branch for a fresh PR. Use when a PR can't merge due to conflicts on a protected destination branch, or the user mentions resolving conflicts between two branches.
---

# Resolving Protected Branch Conflicts

When a destination branch is protected and a PR has conflicts, you can't merge or push directly to it. Resolve the conflict against the **repo's** branch state (origin/), then push the result as a new branch and open a fresh PR. The new branch's parent is the merged state — not the source or destination branch.

Ask for / confirm three values before starting:
- **DESTINATION** — the protected branch the PR targets (e.g. `integration`, `develop`, `qa`).
- **SOURCE** — the feature branch with the changes (e.g. `feature/RCOMM-6102-...`).
- **TICKET** — the ticket ID for the commit message (e.g. `RCOMM-6607`).

## Steps

1. **Fetch latest refs**
   ```
   git fetch
   ```

2. **Check out the repo's destination branch (detached — NOT your local branch)**
   ```
   git checkout origin/DESTINATION
   ```
   This intentionally puts you in a detached HEAD at the repo's current destination state, so the resolution is based on what's actually on the remote, not a stale local copy.

3. **Merge the repo's source branch**
   ```
   git merge origin/SOURCE
   ```

4. **Resolve the conflicts** in each conflicted file. Show the user the conflicting hunks and the proposed resolution; do not guess silently on non-trivial conflicts.

5. **Stage the resolved files** (stage intentionally, by path — not `git add .`)
   ```
   git add PATH/TO/FILE.EXT
   ```

6. **Commit** with the ticket-scoped message (no AI attribution)
   ```
   git commit -m "RCOMM-[XXXX] Resolving conflict between DESTINATION and SOURCE"
   ```

7. **Push the merged state as a NEW branch** — this sets the new branch's parent to the merged state, which is neither the destination nor the source branch
   ```
   git push origin HEAD:refs/heads/SOURCE-with-conflicts-resolved
   ```

8. **Open a new PR** (create only — do not merge it):
   - Source: `SOURCE-with-conflicts-resolved`
   - Destination: `DESTINATION`

   `gh pr create --base DESTINATION --head SOURCE-with-conflicts-resolved`

9. **Get off the repo branch** — check out a normal local branch so you don't keep working in detached HEAD or accidentally on the repo branch
   ```
   git checkout DESTINATION   # or your next working branch
   ```

## Notes
- Steps 2–3 operate on `origin/` refs on purpose; the whole point is resolving against the remote's current state.
- The new `-with-conflicts-resolved` branch is the deliverable — don't force-push the resolution onto SOURCE or DESTINATION.
- Per global CLAUDE.md: review before committing, no `Co-authored-by`/AI attribution in the commit message, and don't `git add .` blindly.
