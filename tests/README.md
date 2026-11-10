# Content-script regression tests

The tests execute the production content scripts in Chromium, with deterministic extension APIs and translation responses. They cover tab-return recovery, bounded language detection, manual-restore preferences, X post expansion, text-node identity, original-text restoration, protected link labels, stale responses, dynamic-content preferences and viewport changes.

From the repository root:

```sh
npm --prefix tests install
npx --prefix tests playwright install chromium
npm --prefix tests test
```

Use `CHROMIUM_PATH` for an existing Chromium executable, or `PLAYWRIGHT_MODULE` when Playwright is installed outside `tests/node_modules`. `TEST_FILTER` selects a test group by part of its name.

This is not an end-to-end test of an installed extension or a live translation provider. Live X/Reddit behavior and Firefox remain manual validation items.

Visibility transitions are dispatched explicitly so lifecycle races are reproducible; they do not simulate native background-tab timer throttling. The normal dynamic-content collection interval remains 2 seconds.


## Bounded Google recovery on X

`retry-regression.cjs` also executes the production background Google adapter,
response parser and cache. Only the remote XHR responses are controlled. It
covers transient transport errors, empty/whitespace/partial/malformed responses,
a cached partial result, the three-attempt limit, two-post recovery concurrency,
success/unchanged responses, hidden/offscreen posts, stale revisions, removal,
manual restoration and sharing pending requests.

`fixtures/twitterTranslator-v1.1.js` is an unmodified historical snapshot of
the X helper from the preceding PR revision (head
`054f524288e5b929ee8ca8d3a94c293bbb5c6d4e`). One comparison test loads that helper
with the current adapter's normal, non-refresh path to reproduce the missing
recovery and cached-error replay. All remaining recovery tests use the current
production helper. No adjacent checkout or network access is needed.

The recovery tests use a controlled Chromium clock, not native timer throttling.
They do not establish the cause of a previously observed intermittent live-X
translation failure. Valid responses that retain the original text and requests
that never settle are outside this recovery mechanism.


## Cached original-text validation on X

`cache-regression.cjs` covers Google cache entries that contain the original
paragraph while reporting a detected language different from the target. The
check is opted into only by X Google HTML requests. It requires at least 80
characters and eight letter-words of unchanged text. Both completed in-memory
entries and persistent entries are checked; newly returned suspect originals
are treated as failures before they can become cached successes.

Unknown/target languages, short names, ordinary pages and valid translations
retain existing behavior. The current two-retry budget and concurrency cap
remain in effect. Tests include text split around a protected mention.

The check is conservative, not proof that every unchanged result is invalid.
It does not detect English paraphrases or entries with unknown source language.
Clearing a live cache removed the evidence needed to establish the exact cause
of that incident; controlled regressions cover the specific class above.
