---
name: wcag-review
description: Accessibility (WCAG) review and fixes for Craft/Twig + Vue/Tailwind/Alpine frontends. Use when the user mentions accessibility, a11y, WCAG, aria, screen readers, focus, keyboard nav, heading order, or announcements, or works a ticket tagged accessibility/WCAG.
---

# WCAG Accessibility Review

A repeatable checklist for the accessibility work I do often (heading order, announcements, focus, keyboard). Apply it when reviewing or fixing a11y on changed markup/components. Review changed code first; only widen scope if asked.

## Checklist

**Structure & semantics**
- Headings descend sequentially (no skipping `h2` → `h4`). Flag e.g. a footer "Mailing Address" rendered as `h4` under an `h2`.
- Use semantic elements (`nav`, `main`, `button`, `ul/li`) over generic `div`/`span` with handlers. A clickable thing that navigates is a link; one that acts is a `<button>`.
- One `h1` per page; landmarks present (`header`, `nav`, `main`, `footer`).

**Keyboard & focus**
- Every interactive element is reachable and operable by keyboard (Tab/Shift-Tab/Enter/Space/Esc).
- Visible focus indicator present; don't remove `:focus-visible` outlines without an equivalent.
- Focus is managed on dynamic UI: opening a modal/menu moves focus in and traps it; closing returns focus to the trigger. Prefer **Alpine UI** primitives (`@alpinejs/ui`, `@alpinejs/focus`) which handle this — don't hand-roll focus traps.

**Screen-reader announcements**
- Error and status messages are announced: associate field errors with inputs (`aria-describedby`), and use a live region (`aria-live="polite"`/`assertive` or `role="alert"`) for dynamic messages. When an error is announced, also move focus to it if it requires action.
- Phrase counts/labels to read naturally for SR users (e.g. "3 incomplete registration(s)" reads better than terse fragments).
- Icon-only controls have an accessible name (`aria-label` or visually-hidden text).

**Forms**
- Every input has a programmatically associated `<label>` (not just placeholder).
- Required/invalid state conveyed via `aria-required`/`aria-invalid`, not color alone.

**Visual**
- Text contrast ≥ 4.5:1 (3:1 for large text); don't rely on color alone to convey meaning.
- Images have meaningful `alt` (or `alt=""` if decorative).

## How to report
1. List findings grouped by the categories above, each with the specific element/file and the WCAG criterion.
2. Propose the minimal fix per finding; prefer Alpine UI / existing components over custom (see global CLAUDE.md: reuse before creating).
3. If fixing, verify the change doesn't regress the design, then offer concise **test instructions** (how a reviewer confirms with keyboard + screen reader).
