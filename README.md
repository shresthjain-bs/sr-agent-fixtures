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
| unlabelled `<input>`s | `MISSING_LABEL` | 4.1.2 | traversal |
| scrambled positive `tabindex` | `BROKEN_FOCUS_ORDER` | 2.4.3, 1.3.2 | LLM focus-order judge |

The page-load findings need neither traversal nor the LLM — that audit runs unconditionally
between navigate and parse-workflow — so the tests pin those hardest. The traversal and
LLM-judged findings depend on the agent tabbing onto the control and on there being enough
navigation events, so they are asserted more loosely.

## Rules

- **Do not "fix" the accessibility of `broken-login.html`.** The defects are the fixture.
- **Do not let `clean-login.html` regress.** A finding there reads as a product regression,
  which is the opposite of what the page is for.
- Both pages are static, self-contained, and carry no credentials or personal data.
