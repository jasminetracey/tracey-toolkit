---
description: Critical code review with findings presented as an actionable plan
argument-hint: [file, folder, commit-ish (abc123 | HEAD~2 | main..HEAD | --last N), or description]
---

Review $ARGUMENTS using a five-reviewer committee process.

If no target is provided, review the files most recently changed in this project.

## Model Selection

Before starting the review, ask the user which model to use for the reviewer agents:

> Which model should the reviewers use?
> 1. **opus** — most thorough, slowest
> 2. **sonnet** — good balance (default)
> 3. **haiku** — fastest, least detailed

Use their choice for all reviewer agents (Phase 2) and peer review agents (Phase 3). Default to sonnet if the user doesn't specify.

## Stack Context

I work across two main stacks. **Detect which one this project uses** before reviewing (check `composer.json`, `package.json`, and the directory layout) and apply the matching conventions. A project may mix both (e.g. Craft + Vue islands), so apply whichever conventions are relevant to the files under review.

**Detection hints:**
- **Craft CMS** — `craftcms/cms` in `composer.json`, a `templates/` dir with `.twig` files, `config/general.php`
- **Laravel** — `laravel/framework` in `composer.json`, `app/Http/Controllers`, `routes/`, `resources/views` (Blade)
- **Inertia** — `@inertiajs/*` in `package.json`, `resources/js/Pages`
- **Filament** — `filament/filament` in `composer.json`, `app/Filament/` (Resources, Pages, Widgets)

Adherence to stack conventions is a first-class concern in every review. Always flag violations of the relevant set below.

### Craft CMS / Twig

- Craft conventions (element queries, services, plugin/module structure, Twig template patterns, eager-loading with `.with()` to avoid N+1)
- Security: `|raw` on untrusted data, missing CSRF tokens on forms, missing permission checks

### Laravel

- Laravel conventions: service classes, form requests for validation, policies/gates for authorization, Eloquent patterns (eager-loading to avoid N+1, mass-assignment guarding), resource controllers, config/env usage over hardcoded values
- Blade conventions where Blade is used (components, slots, no logic-heavy templates)

### FilamentPHP

- Resource/Page/Widget structure and naming; schema definitions in the right lifecycle methods
- Form/Table builder conventions (reusable components, `->rules()` validation, relationship managers over ad-hoc queries)
- Authorization via policies (Filament respects model policies) and `->authorize()`/`->visible()` guards
- Avoid heavy queries in table columns/actions without eager-loading; use `->searchable()`/`->sortable()` correctly

### Vue 3 (both stacks)

- Vue 3 Composition API patterns
- **Inertia.js** where present: correct data-passing from controllers, page component conventions in `resources/js/Pages`, `<Link>`/`router` over raw fetches, shared props via middleware

### House frontend standards (from global CLAUDE.md — apply on any frontend)

- **Reuse before creating** — extend an existing component/partial/field/utility instead of adding a parallel implementation
- **Tailwind utility classes over arbitrary values** (`text-h1`, not `text-[68px]`); define a token once and reference it
- **BEM** naming (`block__element--modifier`) for custom component classes
- **`cx()` helper** for dynamic/conditional classes, not string concatenation or embedded ternaries
- **`useSvgSprite`** convention for SVGs, not inline SVG
- **AlpineJS / Alpine UI components** before hand-rolled JS interactivity
- **Match the design exactly** — exact tokens from Figma, verified against stated breakpoints

## Review Ambition

Reviewers must look for **code judo** moves — restructurings that **delete** complexity rather than rearranging it. Do not stop at "this could be cleaner." Actively search for reframings where whole branches, helpers, modes, conditionals, or layers disappear entirely. Prefer the solution that makes the code feel inevitable in hindsight.

If a path exists to delete complexity rather than reorganize it, push hard for that path. Do not rubber-stamp working code that leaves the codebase messier than it found it.

## Diff-Aware Mode

If the target is inside a git repository, first check whether `$ARGUMENTS` is a **commit-ish** — resolve it and review exactly that diff:

- **Single commit** (a SHA like `abc123`, or `HEAD`, `HEAD~2`, a tag/branch name that resolves to one commit): review the changes introduced by that commit — `git show <ref>` (equivalent to `git diff <ref>~1 <ref>`).
- **Range** (`main..HEAD`, `HEAD~3..HEAD~1`, `abc123..def456`): review the cumulative diff for that range — `git diff <range>`.
- **`--last N`**: review the last N commits combined — `git diff HEAD~N..HEAD`. (`--last 1` is the same as reviewing the most recent commit.)
- Verify the ref exists first (`git rev-parse --verify <ref>` / `git cat-file -t`). If it doesn't resolve to a valid commit-ish, treat `$ARGUMENTS` as a file/folder/description instead.
- Include `git log --oneline` for the reviewed commits in the report header so it's clear what was reviewed.

If no commit-ish and no specific file/folder is given:

- Default to reviewing only lines changed since the last commit (`git diff HEAD`). If there are no uncommitted changes, diff against the previous commit (`git diff HEAD~1`).
- This keeps the review focused on new/modified code rather than flagging old stable patterns.
- If the user passes `--full`, review the entire target without diff filtering.

Even in diff mode, reviewers should read surrounding context to understand the change, but findings should focus on the changed code.

## Review Process

### Phase 1 — Git Blame Context

Before spawning reviewers, run `git blame` on the target files. For each region of code under review, note:

- How old the code is (last modified date)
- Whether it was recently changed or has been stable

Pass this context to the reviewers. Code that has been stable for 6+ months and is not part of the current diff should be flagged with lower confidence — it may be intentional. Reviewers should still flag genuine issues in old code, but should note the age as context.

### Phase 2 — Parallel Independent Reviews

Spawn **5 critical-code-reviewer agents in parallel**, each with the same target code (plus blame context) but a different primary lens:

1. **Security & Correctness** — prioritize vulnerabilities, auth gaps, input validation, logic errors, edge cases, and data integrity
2. **Performance & Scalability** — prioritize query efficiency, memory usage, caching, N+1 problems (Craft element queries in loops, uneager-loaded Eloquent relations, Filament table columns hitting relations per-row), and load behavior
3. **Maintainability & Architecture** — prioritize readability, coupling, convention adherence, test coverage, and design patterns
4. **Bugs & Blunders** — prioritize the mundane mistakes that slip past high-level reviews: typos in variable/method names, copy-paste errors (duplicated lines, wrong variable reused), off-by-one errors, dead code and unreachable branches, wrong comparison operators (`=` vs `==` vs `===`), inverted conditionals, forgotten `return` statements, hardcoded values that should be config/constants, mismatched function signatures, and incorrect string interpolation
5. **Structural Simplification** — apply an unusually strict lens focused on whether the change could be dramatically simpler. Aggressively flag:
    - **Missed code-judo moves** — restructurings that would delete whole branches, helpers, modes, or layers rather than rearrange them
    - **Reuse violations** — a new component/partial/field/utility that duplicates an existing one instead of extending it (per house standards)
    - **Spaghetti growth** — new ad-hoc conditionals, scattered special cases, or one-off branches bolted into unrelated flows
    - **File-size explosion** — diffs that grow a file past ~1000 lines without strong justification (treat as a smell, not a hard blocker)
    - **Thin abstractions** — wrappers, identity helpers, or pass-through layers that add indirection without buying clarity
    - **Magical / hacky behavior** — generic mechanisms that hide simple data-shape assumptions
    - **Boundary sloppiness** — unnecessary optionality, `any`/`unknown`/`mixed`, cast-heavy code, or silent fallbacks papering over unclear invariants
    - **Canonical-layer leaks** — feature logic leaking into shared paths, or bespoke helpers duplicating existing canonical utilities
    - **Avoidable orchestration complexity** — unnecessary sequential async flow, or non-atomic updates where a more atomic structure is obviously cleaner

    Lead findings with structural recommendations that *delete* concepts, not ones that polish them. Be direct and demanding: if the implementation missed a dramatic simplification, say so clearly.

Each reviewer still performs a full review, but leads with their assigned lens. This ensures diverse coverage.

### Phase 3 — Diff & Cross-Validation

After all 5 reviews complete:

1. **Merge** — collect all findings into a single list, deduplicating items found by multiple reviewers
2. **Identify unique findings** — any finding raised by only one reviewer
3. **Peer review unique findings** — for each unique finding, spawn a critical-code-reviewer agent asking: "Reviewer N flagged this issue. Read the relevant code and determine: is this a valid concern, a false positive, or overstated in severity?" Include the finding details and file context.

### Phase 4 — Final Report

Apply the `i-have-adhd` skill's output rules to everything below: no preamble, no closing recap, matter-of-fact tone on errors.

#### Summary Stats

Start the report with a one-line summary, then immediately name the single most urgent fix as its own line — before any table:

> **X critical, Y high, Z low** across N files. M findings disputed.
> **Do first:** fix `file:line` — one-line reason.

If nothing is critical or high, skip the "Do first" line.

#### Findings Tables

Group findings into these confidence tiers:

- **Consensus findings** — issues found by 2+ reviewers (highest confidence)
- **Validated unique findings** — found by one reviewer, confirmed by peer review
- **Disputed findings** — found by one reviewer, disputed by peer review
- **Dismissed findings** — found by one reviewer, dismissed by peer review (list briefly for transparency)

Within each group, order by severity: critical -> high -> low.

Present findings as a markdown table with these columns:

| Severity | File:Line | Issue | Flagged By | Recommended Fix |
|----------|-----------|-------|------------|-----------------|

- **Severity**: critical / high / low
- **File:Line**: file path and line number (e.g. `templates/_components/card.twig:42`)
- **Issue**: concise description of the problem
- **Flagged By**: which reviewer(s) raised it (e.g. "Security", "Perf + Maint", "All 3")
- **Recommended Fix**: brief actionable suggestion

Use a separate table for each confidence group (Consensus, Validated, Disputed, Dismissed). If a table would exceed 5 rows, split it into "Fix now" (critical/high) and "Later" (low) sub-groups instead of one long list.

For **disputed findings**, add a plain-Markdown reasoning block below the table showing both sides. Do NOT use HTML (`<details>`/`<summary>`) — this renders in a terminal that shows raw HTML tags literally. Use this format instead:

**Dispute: [issue summary]**
- **Original reviewer's argument:** ...
- **Peer reviewer's counterargument:** ...

Also flag any reviewed code that lacks tests or would be difficult to test — treat this as a project risk finding.

#### Prioritized Action Plan

After the tables, produce a numbered action plan ordered by impact and dependency:

1. Fix X in `file.twig:42` (critical — blocks deployment)
2. Fix Y in `file.php:88` (high — depends on #1)
3. ...

Group related fixes that should be done together. Skip dismissed findings. Include disputed findings with a note that they're disputed.

#### Auto-Fix Offer

After presenting the full report, ask:

> "Want me to apply fixes? I'll auto-fix consensus and validated findings. Disputed findings will be skipped unless you approve them."

If the user accepts, apply fixes following the action plan order. For each fix, use the recommended fix from the findings table. Skip any fix that would require design decisions or clarification — list those as "manual fixes needed" instead.

## Review Concerns

Beyond correctness, flag any of the following:

- **Complexity** — logic that is unnecessarily convoluted or hard to follow
- **Readability** — code a human would struggle to understand or maintain
- **Project risk** — patterns that could cause bugs, regressions, or failures at scale
- **Security** — pay close attention to: authorization gaps, unvalidated input, exposed sensitive data, SQL injection, XSS, CSRF, mass assignment, and improper use of the framework's security primitives. Craft: `|raw` on untrusted data, CSRF tokens on forms, permission checks. Laravel/Filament: policies/gates, form-request validation, `$fillable`/`$guarded`, and Filament resource authorization

## Preferred Remedies

When suggesting fixes in the **Recommended Fix** column, bias toward deletion-shaped suggestions over polish-shaped ones. Prefer:

- Delete a whole layer of indirection rather than polishing it
- Reframe the state model so conditionals disappear rather than getting centralized
- Collapse duplicate branches into a single clearer flow
- Turn special-case logic into a simpler default flow with fewer exceptions
- Reuse an existing component/partial/field/utility instead of introducing a near-duplicate
- Move feature-specific logic behind a dedicated abstraction, or out of a shared path entirely
- Replace condition chains with a typed model or explicit dispatcher
- Make type boundaries explicit so downstream control flow gets simpler
- Split a large file/template into smaller focused partials
- Separate orchestration from business logic
- Parallelize independent work when that also simplifies the orchestration

Do not settle for "maybe rename this" when the real issue is structural. Do not settle for a cleaner version of the same messy idea when there is a plausible path to a much simpler idea.

## Scope Guidance

If the target is a large directory, focus on the most impactful files rather than exhaustively covering everything. Prioritize files with the most complexity, risk exposure, or security surface area.

## Output

If no significant issues survive the committee process, say so clearly rather than manufacturing minor findings.
