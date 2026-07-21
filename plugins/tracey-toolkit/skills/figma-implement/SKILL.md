---
name: figma-implement
description: Implement a Figma design into this Craft/Twig + Tailwind/Alpine codebase with exact tokens and design-system reuse. Use when a Figma URL is provided alongside a request to build/implement/match a design, component, or screen. Complements the figma plugin's design-context tools.
---

# Implement Figma → Craft/Tailwind

Reduce the design back-and-forth by front-loading exact values and reuse before writing markup. Use the figma plugin tools (`get_design_context`, `get_metadata`, `get_screenshot`, `get_variable_defs`) to read the design; this skill is the implementation discipline around them.

## Workflow

1. **Extract exact tokens, don't approximate.** Pull real values from the design: font sizes, line-heights, spacing, aspect ratios, colors, and any `clamp()`/responsive values. Past friction came from eyeballing — e.g. a clamp that should hit exactly `68px` at 1024px, a background that's `1246×520 / aspect-ratio 127/53`. Get the number from Figma, then verify the rendered result matches at the stated breakpoint.

2. **Map tokens to the design system, not arbitrary values.** Prefer existing Tailwind tokens/utilities (`text-h1`, `--color-secondary-yellow-500`) over arbitrary values (`text-[68px]`). If a value appears in more than one place, define it once as a variable and reference it (don't duplicate, e.g. the same size in both `text-body-lg` and `prose-xl`).

3. **Reuse before creating.** Search the codebase for an existing component/partial/util that already renders this pattern and use/extend it. Do not spin up a parallel implementation (the recurring "why did you create X instead of reusing Y" problem). If a genuinely new component is warranted, say why before building.

4. **Honor the responsive intent.** Check each breakpoint in the design. Keep containers/grids intact on mobile unless the design explicitly removes them; confirm text doesn't get cut off or overflow. When the designer's note is ambiguous ("line animates across as user scrolls"), state your interpretation and confirm before building.

5. **Conventions** (see global CLAUDE.md): BEM for custom component classes, `useSvgSprite` over inline SVG, Alpine/Alpine-UI for interactivity before custom JS.

6. **Verify, then ship.** Compare the build against the Figma screenshot at the key breakpoints. Note any intentional deviations. Then hand off (or `/ship`) with test instructions.

## Anti-patterns to avoid
- Arbitrary Tailwind values when a token exists.
- Duplicating a value across rules instead of a shared variable.
- New component/field when an existing one (or a Custom Link Type) fits.
- Dropping the container/full-bleeding on mobile when the design keeps it.
