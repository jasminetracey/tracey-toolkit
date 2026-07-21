---
description: Review changed files, commit (ticket-scoped), push, and open a PR to the target branch with test instructions
argument-hint: "[ticket/issue e.g. #264 or RCOMM-6607] [target branch e.g. qa|integration|develop]"
allowed-tools: Bash(git *), Bash(gh *), Skill(review), Skill(review:*)
---

Ship the current work: review, commit, push, and open a PR. Arguments (optional): `$ARGUMENTS` — first token is the ticket/issue reference, second is the target branch.

Follow these steps exactly. **Never commit work I have not reviewed** (per global CLAUDE.md), so the review + confirmation gate below is mandatory.

## 1. Survey
- `git status` and `git diff` to see what changed. Also `git log --oneline -5` for recent commit-message style on this repo.
- Identify the **ticket/issue**: use the one in `$ARGUMENTS` if given; otherwise infer from the current branch name (e.g. `feature/RCOMM-6102-…`, `feature/859-…`) and confirm with me.
- Identify the **target branch**: use `$ARGUMENTS` if given. Otherwise determine the repo's integration target — check for `qa`, `integration`, `develop`, `v2`, or the project's rebuild branch (`git branch -a`) and **ask me which one** if it's ambiguous. Do not assume `main`.

## 2. Review (this is the gate — do not skip)
- Review **only the changed lines**, not pre-existing code. (I've repeatedly had to say "review only changed stuff, not existing stuff" — respect that.)
- Run the `review` command (five-reviewer committee process) on the diff. It runs in diff-aware mode by default, so it will focus on the changed lines.
- Surface anything I should look at, applying the `i-have-adhd` skill's rules: lead with the action needed (fix something first, or clear to commit), not a recap of what the review checked.
- Then **show me the staged diff summary and stop for my confirmation** before committing.

## 3. Commit
- Stage intentionally — do **not** `git add .` blindly. Exclude `CLAUDE.md`, `plans/`, and `.planning/` (global rule).
- Message format: **ticket number first, then imperative subject**, e.g. `#264 Add SEO to text page`, `RCOMM-6607 Implement new nav design`. Keep it scoped to one logical change; split into multiple commits if the work spans multiple tickets.
- **No attribution trailers.** Never add `Co-authored-by`, "Generated with Claude", or any AI attribution to the commit message or PR body. All work is authored solely by me.

## 4. Push & PR
- Push the current branch.
- Open the PR against the target branch from step 1 with `gh pr create`.
- Keep the body lean — developers review these; skip redundant or obvious detail. Use exactly this shape:

  ```markdown
  ## Summary

  - <bullet points, not paragraphs — one per logical change>

  **Ticket:** #NNN

  ## Test

  1. <simple, copy-pasteable steps a reviewer/QA follows to verify>
  ```

  Rules for the shape: **Summary** is bullet points only (never a prose paragraph); the ticket goes on its own line immediately below the Summary as `**Ticket:** #NNN` (not its own `##` heading); **Test** is numbered, copy-pasteable steps — always include them, I ask almost every time.
- Return the PR URL.

## Notes
- If I only say `/ship` with no work staged yet, ask what to include.
- If a PR already exists for this branch, push to update it instead of creating a duplicate, and summarize what changed.
