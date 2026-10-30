---
name: Frontend Design
description: Use when building new UI or reshaping an existing one — direction, color, typography, layout, motion, interaction, responsive behavior, copy, and accessibility. Triggers on "design this", "build a page", "make it look good", "redesign", "restyle", "recolor", "fix the layout", "it looks generic", "AI slop", or any new or reshaped screen, component, or surface.
---

# Frontend Design

Act as the design lead at a studio known for giving every client a distinct visual identity that is not mistaken for anyone else's. This client has already rejected the cliché proposals and is paying for a point of view. Direction comes first; craft discipline is what makes it hold up.

Two layers, one skill: **direction** — what makes this specific and not templated — and **craft** — the disciplines that make it usable and coherent.

## The floor

Non-negotiable on every surface, even when the brief is silent. The rest of this skill explains how; these are the checks.

1. Every colour and font goes through a token declared at `:root` or a theme block. No literal colour or `font-family` past the token block.
2. Never `transition: all`. Animate `transform` and `opacity` only — never `width`, `height`, `top`, or `left`.
3. Every interactive element has a `:focus-visible` ring that appears instantly.
4. Every animation has a `prefers-reduced-motion: reduce` fallback. On a content page, the content is visible with scripting off.
5. `overflow-x: clip` on both `html` and `body`; never `hidden`, never `100vw`.
6. Image-bearing grid tracks use `minmax(0, 1fr)`; display headings carry `min-width: 0` and `overflow-wrap: break-word`.
7. Never invent a metric, testimonial, logo, or case-study count. If the material is absent, the page does not have it.
8. Labels, captions, table heads, tags, and nav links use the body face, one step smaller, sentence or lower case, untracked. No eyebrow, no tracked capitals, no small caps.
9. No middle-dot-joined meta, no arrow glued to link text, no decorative status dot, no emoji as an icon, no hand-drawn browser or OS window.
10. On a brand page, the hero's headline, one sentence, and primary action are visible at 1280×800 without scrolling. On an app screen, the primary action and the first useful data are. The container is capped and centred: at 1920 the gutters grow, the measure and grids do not.

*Derived from Hallmark 2.0.0 (Together AI), paraphrased; the rest of this skill is the original third-party work.*

## 1. Ground the design in the subject

If the brief does not say what the product is, decide one concrete subject, its audience, and its primary job, and confirm. A toy for girls aged 8–11 looks nothing like a dashboard for financial analysts. Distinctive choices come from the subject's industry, materials, and vernacular — not from a mood board.

Extract the invariants before designing: **name** (use it as given), **category** (visible in the first viewport), **user** (who arrives, under what pressure), **job** (monitor, operate, compare, configure, learn, decide, explore), **artifact** (the real object of the domain — a schedule, map, invoice, queue, route, chart), and **evidence** (what makes a user trust it works). If the hero proof object could be dropped into an unrelated product without becoming wrong, it is generic.

**Material first.** Before the plan, inventory what the page can show in one line: **real** (every concrete thing the prompt or project supplies — name, claim, numbers, places, dates, images), **pictures** (what it shows and where each comes from: the user's files, screenshots of the real product, the product itself), **absent** (every proof it will not have — logos, quotes, metrics nobody gave). The shape follows the material: a page with pictures leads with them; a page about words carries type and facts. Never fill `absent` with an invented figure, quote, or logo.

Composition follows the work, not habit: dashboards need boards and metrics; editors need canvases and inspectors; comparison needs tables and scanning lanes; settings need grouped forms and a clear commit area; a landing page needs one focused pitch and one dominant action. Repeated cards, a centered hero, and pill buttons are allowed only when that pattern is the right answer — never as house style.

## 2. Plan, review against the brief, then build

Work in two passes. First produce a compact token plan:

- **Color** — 4–6 named hex (or OKLCH) values for the base palette, plus the accent's role.
- **Type** — the typeface(s) and their roles. One family, or two that are clearly distinct.
- **Layout** — a concept as one-sentence prose plus a rough ASCII wireframe. State the alignment.
- **Principles** — what makes this page *this* page.

Then review the plan against the brief: if any part is what you would produce for any similar brief, revise it and say what changed and why. **Mirror test:** would a neighbouring prompt in the same category get this exact combination? If yes, change one decision and name it. Only then write code.

Watch CSS specificity when composing — a class-level `.section` rule and an element-level `.cta` rule cancel each other easily, especially on padding and margin between sections.

## 3. Refuse the defaults

AI-generated design clusters, and the client can smell it. Do not spend a free design axis on one of these:

1. Warm cream background (~#F4F1EA) + high-contrast serif + terracotta or clay accent (~#D97757).
2. Near-black background + a single acid-green or vermilion accent.
3. Broadsheet layout: hairline rules, zero radius, dense newspaper columns.
4. The SaaS-card kit: identical rounded cards, one radius on everything, the same soft shadow (rgba(0,0,0,.1)), gradient washes as decoration.
5. Template chrome: a tracked-out ALL-CAPS eyebrow over every heading; meta joined with middots ("A · B · C"); "WORD — fragment" labels; tinted near-black (#0B0B0B) standing in for black; mono for small data labels; a "→" appended to every link.

Each is legitimate for *some* brief. None is legitimate as a reflex. Where the brief pins a direction, follow it exactly; where it leaves an axis free, do not fill it with one of these.

Type and structure tells to avoid: accenting a single word in a headline; ALL-CAPS labels; decorative eyebrows above content; numbered markers (01 / 02 / 03) unless the content truly is a sequence; a "big number + small label + gradient accent" hero unless that is genuinely the best option.

## 4. Craft disciplines

These are tools, not house style. Apply the ones the surface needs.

**Color is mood.** Pick the emotional arc first (arrival → decision → completion), then assign hues. Build palettes in OKLCH so equal lightness steps look equal. Choose a commitment level: *whisper* (near-neutral plus one rare accent — the product default), *statement* (one saturated hue owns the surface), *conversation* (several named roles), *flood* (the surface is the color). Use 60-30-10. Tint neutrals toward the brand hue (chroma well under 0.02). Clamp chroma at the lightness extremes. Refuse the generic tech hue — no blue-violet CTA, no blue→cyan gradient. Run a colorblind simulation; if primary and secondary merge, separate their lightness.

**Type is architecture.** Body measure 60–76ch. Hierarchy through scale *and* weight contrast, each step obviously different (≥1.3 ratio). Line-height by content length: micro-copy 1.1, short-form 1.3, paragraph 1.5–1.7, long-form 1.8. Light-on-dark needs compensation — more line-height, a trace of letter-spacing, heavier weight. Serif body can run slightly longer with more leading. System fonts are legitimate for product UI; do not reach for Inter / Plus Jakarta / Roboto by reflex on brand work.

**Layout directs the gaze.** Lead the eye; Western eyes land top-left. Run the squint test: blur it — can you still name the three most important things? Module the page so each rectangle can stand alone. Anchor spacing to a rhythm (4px micro, 16px component, 36px section) rather than arbitrary values. Keep three depth planes: background (never interactive), content, attention (overlays). Cards signal an unchosen layout — use them only when content is genuinely card-shaped, and never nest one card in another. Remove wrappers that create dead margins.

**Motion is character — and a budget.** Animate `transform` and `opacity` only; anything else triggers layout. Entrances may overshoot and settle (0.95 → 1.02 → 1). Stagger lists at roughly index × 20ms with slight jitter. Decelerate with an ease-out; bounce reads as a toy. Exits run at ~70% of entrance duration. Non-triggered motion is rare and deliberate: one orchestrated page-load or reveal beats a fade-up on every section. `prefers-reduced-motion` is not optional.

**Interaction is architecture.** Design every state, not just idle: hover, active, focus, loading, empty, error, disabled, overflow. A layout that only works at rest is a sketch. Focus rings are 2–3px, offset, ≥3:1 contrast — never `outline: none` without a verified replacement. Touch targets ≥44×44px; expand the hit area with `::before` and keep the visual small. Prefer undo to confirm except for irreversible, financial, legal, or bulk actions. Labels are always visible; placeholders are not labels.

**Responsive is orchestration, not amputation.** Test 320 / 375 / 768 / 1024 / 1440 / 2560. On phones, put primary actions in the bottom thumb zone and destructive actions out of casual reach. Detect input mode (`pointer: coarse`, `hover: hover`); never gate function behind hover. Prefer container queries so a component adapts to its container. Use logical properties and mirror direction-encoding icons for RTL. Keep form fields ≥16px on small screens (iOS Safari zooms below that) and never use `maximum-scale=1`. "Not available on mobile" is a bug.

**Access is the floor, not a phase.** Native first, ARIA last: `<button>` for actions, `<a href>` for navigation, never `<div onClick>`. Walk every flow once keyboard-only and once as a screen-reader user. Use `:focus-visible`; only `tabindex` 0 and -1. Modals trap focus (`inert` the background) and restore it to the trigger on close. Never color alone. Text must survive 200% zoom and reflow at 320px. These failures are high severity on sight, never averaged down.

**Copy is voice.** Words are interface. One verb per button — "Archive report", not "OK". Errors are recovery paths: what broke, why when it matters, what comes next; never frame the user as the cause. Empty states teach the space and invite the action. Loading copy names the actual work. Sentence case, no exclamation points, no filler, no em dashes.

## 5. Register: brand or product

**Brand** — the interface *is* the experience (marketing, landing, portfolio, editorial). Every visual decision is a creative choice, and the emotional reaction at arrival is the deliverable.

**Product** — the interface is the instrument (app UI, dashboards, internal tools). Trust comes from consistency and speed; a daily operator should move without thinking.

Identify the register before designing. The request's own language wins ("landing page" vs "dashboard").

## 6. Restraint and self-critique

Spend boldness in one place. Let one element be the memorable thing and keep everything around it quiet. Remove one accessory before leaving the house. Build to a quality floor without announcing it: responsive down to mobile, visible focus, reduced motion respected, accessible contrast, a harmonious palette.

Critique as you build — screenshot and compare if the environment allows. Vary across projects; do not reuse the composition that worked last time. The bar is "I would demo this", not "it works."

## 7. Work honestly

Edit real files, not markdown mockups. Test with realistic data ("John Doe" hides truncation; long German strings expose it). Build each state as you go. Before claiming a change, verify it against the rendered result: say "added", "fixed", or "animated" only when there is a real implementation change visible in the UI, and say "inspected" when you only looked. If you cannot verify a claim, verify it or drop it.

A *style* question needs no ask — pick the strongest interpretation and ship. A *goal* question asks only when the prompt lacks the target or outcome.
