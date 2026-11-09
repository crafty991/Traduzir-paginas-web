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
