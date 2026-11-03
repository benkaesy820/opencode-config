# Browser Bridge — issues from agent use

One entry per friction met while using the bridge for real work. This is not a
test report and not a feature list: it is the cost the tool put between an
agent and a verified change. Each entry says what I tried, what happened, why
it cost, and the smallest thing that would help.

Context: OpenCode v2.2.3, the 13-tool MCP surface, Chromium on Linux, working
a React storefront on localhost.

## Sessions

**1. A failed script orphans the session (2026-11-01).**
Inside one `execute` block I called `browser_start`, then a later call failed;
the block returned no id. There is no session list, sessions do not time out,
and `browser_close` needs the id, so the browser stays open. I lost two this
way. *Help:* a `browser_sessions` list with close, or an idle timeout, or a
start result that survives a throw.

**2. Session ids are the only handle, and not namespaced (2026-11-01).**
After the above I used a guessed id and got `SESSION_NOT_FOUND` with no way to
ask what exists. *Help:* the same list.

## Evaluation

**3. `browser_js` does not await a returned promise (2026-11-01).**
`(async () => { await wait(400); return {…} })()` resolves to `{}`; the same
expression written synchronously works. Any "click, wait, read" script has to
be split across calls or forced into synchronous XHR. *Help:* await thenables
before serialising, with the current timeout semantics.

**4. `screenshot: false` while an image is attached (2026-11-01).**
`browser_observe({ includeScreenshot: true })` arrived with the image attached
but no field saying it existed or where it is. I read `observation.screenshot`
and got nothing. *Help:* a path or a boolean in the structured result, so a
script can find the file instead of parsing the message.

**5. Page-initiated navigation did nothing (2026-11-01, twice).**
`location.href = "/"` and `history.back()` inside `browser_js` left the URL,
the title and the scroll untouched; `browser_act({ type: "navigate" })` worked
immediately. It may be the isolated world or a deliberate guard. *Help:* say
so in the docs, or allow it behind the same navigation path `act` uses.

## Targeting

**6. Refs go stale after any intervening call (2026-11-01).**
An `observation` ref is rejected as stale once a `browser_js` or another
observe has run, even a read. Name+role targeting works around it, but only
when the name is unique: a page with a header "Sign in" and a body "Sign in",
or a field's clear button and the empty state's clear button, is ambiguous and
the tool refuses. *Help:* a scope (`within: "dialog" | "main" | "header"`), an
index, or name resolution that prefers the visible/active region.

**7. A delta observation omits controls that just appeared (2026-11-01).**
After clicking "Sign in" the dialog opened, but `mode: "delta"` listed none of
its controls; `mode: "visual"` did. *Help:* define delta as changed *and*
newly visible controls, or point at the visual observation when a new layer
appears.

**8. A disabled-control refusal names an already-stale ref (2026-11-01).**
`TARGET_NOT_ACTIONABLE` returns the current candidates, but retrying the ref
it names fails as stale. Minor: it costs one more observe.

## Environment

**9. No viewport on a session (2026-11-01).**
`browser_start` has no size; the window cannot be resized. Every 320/390 check
needs a second headless browser with its own login state, which cannot be
authenticated the same way. *Help:* `viewport: { width, height }` (and
`colorScheme`) on start, or a viewport tool per tab.

**11. Expected console noise is reported every time (2026-11-01).**
A signed-out app 401s `/auth/me`; every action then warns "page console error:
401". It is the normal state, and it trains me to ignore warnings. *Help:* a
per-session ignore pattern, or once per unique message.

## What worked well

Name+role targeting, `browser_batch` with its `wait`, `browser_storage` for
cookies, the visual observation alongside an action, and `browser_read` for
the page's text all behaved predictably; most of the cost above is in getting
a session and a target, not in the page work itself.

## Status after the 2026-11-01 rebuild

Twelve issues, all fixed except the one that needs a new concept (#6).

| # | State |
| - | ----- |
| 1, 2 | **Fixed.** `browser_tabs` with no `sessionId` lists every live session with its `client`, `paused`, `idleMs` and tabs. A session whose client is gone is closed after a fifteen-minute idle lease, so the lost-session and never-reaped cases both end. |
| 3 | **Fixed.** `browser_js` awaits a returned promise, bounded at ten seconds. An async IIFE returns its value. |
| 4 | **Fixed.** `artifacts[0].path` names the PNG; `observation.screenshot` still never existed. |
| 5 | **Fixed.** A script that navigates is waited for and reported as `navigation`; the result also carries the fresh observation. |
| 6 | **Partly fixed.** A ref now survives a read, a wait and an action on an unchanged page — it dies only when the control set changes, or on `resume`. What remains is the *ambiguity* half: two controls with the same name in different regions (a header "Sign in" and a dialog "Sign in") still refuse, because there is no way to say "the one in the dialog". That needs a `within` scope, which is new surface, not a fix. |
| 7 | **Fixed.** Delta keys on element identity, not the label: a dialog whose fields reuse the page's labels is reported as added. |
| 8 | **Fixed.** The disabled refusal no longer costs the ref its life, and a wait on a genuinely stale ref fails `STALE_OBSERVATION` instead of burning its timeout. |
| 9 | **Fixed.** A viewport can be set at `browser_tabs`; on a compositor whose window cannot reach the requested size the page is pinned to it instead, so a 390 preset measures 390. |
| 10 | **Fixed.** A repeated console message reports once as `page console error: … (×2)` and is never re-reported on a later call. |
| 11 | **Fixed.** `browser_read` with `save: true` writes the page to the session directory and returns an excerpt plus `page.saved.path`. |
| 12 | **Fixed.** The ownership bar takes no pointer events (only its own control does), and it collapses at phone width. A control underneath it clicks through, verified in a headed test that fails without the change. |

Still open: #6's ambiguity scope, and nothing else from this list. There is no
`browser_feedback` report outstanding.

