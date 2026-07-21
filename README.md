# tracey-toolkit

Personal Claude Code commands and skills, packaged as a plugin so they stay in
sync across all my devices from a single source of truth.

This repo is **both a marketplace and a plugin**:

```
tracey-toolkit/
├── .claude-plugin/
│   └── marketplace.json        # marketplace catalog (@tracey-personal)
└── plugins/
    └── tracey-toolkit/
        ├── .claude-plugin/
        │   └── plugin.json     # plugin manifest
        ├── commands/           # /tracey-toolkit:review, /tracey-toolkit:ship
        └── skills/             # figma-implement, resolve-protected-branch-conflicts, wcag-review
```

## Install on a new device

```bash
claude plugin marketplace add jasminetracey/tracey-toolkit
claude plugin install tracey-toolkit@tracey-personal
```

## Update to the latest version

Pushes to `main` are picked up on update (the plugin is auto-versioned by git commit):

```bash
claude plugin marketplace update tracey-personal
claude plugin update tracey-toolkit@tracey-personal
```

## Editing

Edit files directly in this repo, commit, and push. On other devices, run the
update commands above to pull the changes.

## Contents

### Commands
- **`/tracey-toolkit:ship`** — review, commit (ticket-scoped), push, open a PR with test instructions.
- **`/tracey-toolkit:review`** — critical five-reviewer committee code review, stack-aware (Craft/Twig, Laravel, Vue/Inertia, Filament).

### Skills
- **figma-implement** — implement a Figma design into a Craft/Twig + Tailwind/Alpine codebase.
- **resolve-protected-branch-conflicts** — resolve merge conflicts on a protected branch via a fresh PR.
- **wcag-review** — accessibility (WCAG) review and fixes for Craft/Twig + Vue/Tailwind/Alpine frontends.
