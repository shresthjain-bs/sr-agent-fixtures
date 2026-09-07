# SR Agent test fixtures

Static pages used by the BrowserStack AI **SR Agent** (Experience Agent) scan tests in
`browserstack/BstackAIAutomation` — `api/src/features/a11y/weba11y_sr_agent_scan.feature`.

Served over **GitHub Pages** rather than `raw.githubusercontent.com`, because raw returns
`text/plain`: an `.html` there renders as source, not as a page a screen reader can
traverse. The SR Agent drives a real browser session from the service side, so the target
must be reachable from the public internet and unauthenticated — which rules out the
internal `qa-live-server-internal.bsstag.com` host that the healing fixtures use (it
resolves to private `10.72.x.x` addresses and returns 401 on every path).

| Page | Purpose |
|---|---|
| [`broken-login.html`](broken-login.html) | Pinned accessibility defects, mapped to report `violationType`s |
| [`clean-login.html`](clean-login.html) | The accessible counterpart — should yield a zero-violation report |

## Defects planted in `broken-login.html`

| Defect | Expected `violationType` | WCAG | Detected by |
|---|---|---|---|
| blank `<title>` | `PAGE_TITLE` | 2.4.2 | page-load audit |
| no skip link | `SKIP_LINK` | 2.4.1 | page-load audit |
| icon-only `<button>` | `MISSING_LABEL` | 4.1.2, 1.1.1, 2.4.6 | traversal |

### Keep these fixtures minimal

One control, one interaction, no form and no typing. The first revision was a two-field
login form and the agent never finished it: it could not move focus out of the browser
address bar and burned all 900 actions (`MAX_TOTAL_ACTIONS`) without producing a report, so
the report contract went untested entirely. The fewer turns the agent has to take, the more
likely a scan reaches finalize. Add complexity only once the agent is reliable enough to
earn it.

The two page-load defects (blank title, missing skip link) are the ones worth leaning on:
that audit runs unconditionally between navigate and parse-workflow, so it needs neither
traversal nor the LLM and is unaffected by the agent getting lost.

### An accessibility defect is not an automation blocker

Every control on `broken-login.html` keeps a `name`/`id` that automation can target. A real
unlabelled input still has `name="username"`; what makes it inaccessible is having no
`<label>`, `aria-label` or accessible name, so NVDA announces a bare "edit". An earlier
revision stripped the `name`/`id` too, which made the form unreachable - the agent looped on
fallback clicks (`input[name='username']`, `input#username`, `input[placeholder*='user']`)
about every 9s until it exhausted its action budget, and never produced a usable report.
**Keep those attributes.**

Positive `tabindex` scrambling was also removed: it fought the agent for control of
traversal and cost more than the `BROKEN_FOCUS_ORDER` finding was worth. A focus-order
fixture belongs on its own page.

The page-load findings need neither traversal nor the LLM — that audit runs unconditionally
between navigate and parse-workflow — so the tests pin those hardest. The traversal and
LLM-judged findings depend on the agent tabbing onto the control and on there being enough
navigation events, so they are asserted more loosely.

## Rules

- **Do not "fix" the accessibility of `broken-login.html`.** The defects are the fixture.
- **Do not let `clean-login.html` regress.** A finding there reads as a product regression,
  which is the opposite of what the page is for.
- Both pages are static, self-contained, and carry no credentials or personal data.
