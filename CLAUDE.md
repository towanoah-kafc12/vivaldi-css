# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a static Vivaldi UI custom-CSS repository. `main.css` is the active stylesheet loaded into Vivaldi; there is no application runtime, package manager, build step, linter, or automated test suite in this repository.

- `main.css` — active rules for the collapsed/hover-expanded vertical tab overlay and its stacking interactions.
- `docs/vivaldi-custom-css.md` — source of truth for the intentional layout, Vivaldi 8.1 selector findings, layering constraints, and manual verification steps. Read this before changing `main.css`.
- `bk/` — historical CSS snapshots for comparison only; do not treat them as active stylesheets.

## Commands

There are no project build, lint, test, or single-test commands.

```bash
# Inspect the intended changes (ignore the repository's CRLF line-ending noise)
git diff --ignore-space-at-eol -- main.css docs/

# Check the diff for whitespace errors while ignoring CRLF differences
git diff --ignore-space-at-eol --check

# Inspect current working-tree changes
git status --short
```

Visual verification requires reloading the custom CSS or restarting Vivaldi. Use the checklist in `docs/vivaldi-custom-css.md` after every UI change.

## CSS Architecture and Constraints

The vertical tab containers (`#tabs-container` and `#tabs-subcontainer`) are absolute overlays. They are `32px` wide normally and expand to `150px` on hover. Their high `z-index: 99999999` is intentional: lowering it makes the expanded portion appear hidden behind other Vivaldi UI even though its computed width changes.

Instead of lowering the tab overlay or offsetting it from the top, elevate only the UI that must sit above it:

- `.auto-hide-wrapper.top` — keeps the auto-hidden address bar and Vivaldi menu icon interactive.
- `#browser .button-popup:has(.WorkspacePopup)` — keeps only the workspace picker above expanded tabs.

Do not replace these with a `top` offset on the left tab containers. That workaround creates a visible blank strip above the tabs. Scope future high-`z-index` rules to the smallest Vivaldi UI component possible; raising generic popup selectors can affect unrelated controls.

Vivaldi UI selectors are internal and can change after a browser upgrade. If a customization regresses, inspect the installed Vivaldi `resources/vivaldi/style/common.css` and update only the selectors in `main.css`; do not modify Vivaldi installation files.
