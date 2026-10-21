---
name: Design Audit
description: Use when reviewing UI, UX, or design quality — a screen, a flow, a component, an app, or a UI change — usability, accessibility, states, responsive behavior, copy, consistency, and design thinking. Triggers on "audit the UI", "UX review", "design review", "is this usable", "check accessibility", "review this page", "does this look good", or a UI change before it ships. Subject skill for `deep-audit` on interface work; uses the browser-bridge instrument for evidence where available.
---

# Design Audit

Interfaces fail differently than code: states that were never designed, flows no one can finish, text no one can read. None of that is visible in a code read. Measure what can be measured; label taste as taste.

## 1. Frame it

- Which screens and flows are in scope; who uses them and under what pressure.
- Register: **brand** (the interface is the experience) or **product** (the interface is an instrument). The quality bar differs.
- Reviewing a change? Capture before and after at two viewports — that diff is the scope. Whole-app review? Inventory everything below.

## 2. Inventory

The coverage ledger for UI: screens × states × breakpoints × input modes. Every cell gets a verdict: finding / clean / not checked.

- **States:** idle, loading, empty, error, disabled, overflow, first-run — per interactive surface.
- **Breakpoints:** 320 / 375 / 768 / 1024 / 1440 / 2560.
- **Input modes:** mouse, touch, keyboard, screen reader.
- **Themes:** light and dark, if both exist.

## 3. Evidence instruments

- **browser-bridge** (when available): viewport presets and custom sizes, screenshots, click-through, keyboard presses, form entry with edge data, diagnostics (console, failed requests).
- **Static:** templates and components (which states are coded?), CSS (tokens actually in use), markup (labels, landmarks, alt text).
- **Realistic data:** long strings, huge numbers, unicode, empty. "John Doe" hides truncation; long German strings expose it.
- A screenshot you did not take is not evidence. State what you did not check.

## 4. Lenses

**System and variety** — one product is one system: palette, type, components, spacing, voice, and state patterns stay constant across pages. Composition is per page and follows its job: a monitoring board is dense, a checkout is focused, settings are calm, an editor is canvas-first. Check both failure directions — drift (two pages that look like two different products) and template sameness (every page repeats one layout, or a page would slot into an unrelated product unchanged). Drift is measured; sameness is usually judgment — label it.

**Craft and consistency** — count the colors, radii, and spacings actually in use; drift from the system is a finding. Hierarchy via the squint test: blur a screenshot — can you still name the three most important things? Alignment and spacing rhythm.

**States** — every interactive surface has idle / loading / empty / error / disabled / overflow designed and rendered. A missing state is a finding, not a nit.

**Usability** — walk the primary job end to end as a first-time user. Friction, affordances, error recovery ("what broke, what comes next"), destructive-action treatment, empty states that teach.

**Interface versus schema** — the control follows the job, not the storage. Booleans are toggles, small choices radios or segmented controls, longer ones selects, numbers bounded inputs, collections add and remove rows — never a raw JSON, YAML, or config dump standing in for the setting it happens to store. No internal vocabulary where the user reads: schema keys, snake_case, type names, `null`, UUIDs as labels, "invalid JSON" as an error. Raw editors are an escape hatch for experts — labeled, validated, never the default. A user afraid of making a mistake is a defect: constrain the input, preview before apply, prefer undo to confirm, and never punish a typo with data loss.

**Accessibility** — `frontend-design` §4 holds the standards; audit them by walking every flow keyboard-only, then measuring contrast, focus visibility, zoom and reflow, focus trap and restore, reduced motion, and color-only signalling. Never eyeball a contrast value.

**Responsive** — `frontend-design` §4 holds the standards; verify at the documented breakpoints: no horizontal scroll, primary actions reachable on a phone, no function behind hover, fields ≥16px on small screens.

**Copy** — one verb per button; errors say what broke and what comes next; loading copy names the work; sentence case; no filler.

**Design thinking and product fit** — the first viewport says what this is; the composition follows the job (board, editor, table, form); does the interface have a point of view or is it house style; would the hero proof object survive in an unrelated product?

## 5. Evidence classes

- **Measured** — contrast values, target sizes, tab order, overflow at 320px, a state missing from the code: findings with evidence.
- **Judgment** — composition, identity, genericity, tone: recommendations, clearly labeled, never dressed as defects.

## 6. Severity

`code-review`'s scale, unchanged: Blocker, Should fix, Consider, Nit. The UI anchors: a Blocker is a flow that cannot be completed or an action impossible by keyboard; Should fix is a missing state, failing contrast, hover-gated function, broken responsive behavior, or a misleading error; Consider is hierarchy, consistency drift, and copy.

## 7. Report

```text
<verdict>

Findings — <severity order; each: screen/state, what breaks, trigger, evidence (screenshot path or file:line)>

Recommendations — <judgment calls, clearly separated>

Coverage — <screens × states × breakpoints: finding | clean | not checked>

Unverified — <what could not be checked and the exact steps to check it>
```

## 8. Without the browser half

If browser tools are unavailable, or the app is not running and you cannot start it: do the static half — states in code, markup accessibility, token scan, copy — and list every visual and interactive check as `Unverified` with the exact steps for the caller. Never claim a visual result you did not see.

## Rules

- Report, do not restyle. This is an audit.
- Never report taste as a defect.
- Measure contrast and sizes; do not estimate them.
- Check both: the state exists in code, and it renders correctly.
- Walk every flow keyboard-only before signing off.
- Evidence over adjectives.
