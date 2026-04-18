# Examples

Drop-in workflow files. Copy the one that matches your stack into `.github/workflows/`.

| File | What it does |
|------|--------------|
| [`basic.yml`](basic.yml) | Just the QA Advisor PR comment. Zero deps beyond `actions/checkout`. |
| [`with-jest-reporter.yml`](with-jest-reporter.yml) | PR comment + Jest test runs posted to the QualityPilot dashboard via `@qlens/jest-reporter`. |
| [`with-playwright-reporter.yml`](with-playwright-reporter.yml) | PR comment + Playwright runs (with flaky-on-retry tracking) via `@qlens/playwright-reporter`. |
| [`with-pytest-reporter.yml`](with-pytest-reporter.yml) | PR comment + pytest runs via `qlens-pytest-reporter`. Auto-registers via `pytest11`, no `conftest.py`. |
| [`quality-gate.yml`](quality-gate.yml) | Fails CI when QA Advisor flags the PR HIGH risk — block merges until a senior reviewer signs off. |
| [`group-by-file.yml`](group-by-file.yml) | v1.3.0+ — scenarios grouped by sourceFile (with priority badges per row) instead of by must/should/nice. |

## Reporters

- npm: [`@qlens/jest-reporter`](https://www.npmjs.com/package/@qlens/jest-reporter), [`@qlens/playwright-reporter`](https://www.npmjs.com/package/@qlens/playwright-reporter)
- PyPI: [`qlens-pytest-reporter`](https://pypi.org/project/qlens-pytest-reporter/)
- API key: get one at [qlens.dev/dashboard/keys](https://www.qlens.dev/dashboard/keys) (Pro+ plans), set as `QLENS_API_KEY` repository secret.

## Need something else?

Full setup guide: https://www.qlens.dev/docs/install
API schema: https://www.qlens.dev/docs/ci-ingest
