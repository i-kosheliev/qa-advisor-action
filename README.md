# QA Advisor — GitHub Action for PR Test Coverage Analysis

Automatically analyzes PR diffs and comments with test scenarios reviewers should check. Detects **22 change patterns** (auth, payments, env vars, GraphQL schemas, file I/O, date/time, etc.) and classifies each as `must-test`, `should-test`, or `nice-to-test`.

Local heuristic — **no LLM calls, no API keys, no external services**. Runs instantly in CI.

[![Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-QA%20Advisor-brightgreen?logo=github)](https://github.com/marketplace/actions/qa-advisor-pr-test-coverage-analysis)
[![Release](https://img.shields.io/github/v/release/i-kosheliev/qa-advisor-action)](https://github.com/i-kosheliev/qa-advisor-action/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Example PR comment

> ## QA Advisor: Test Coverage Analysis
>
> **Risk Level: MEDIUM** 🟡 | 3 files changed (+142/-28 lines)
>
> ### Must Test (2)
>
> | Scenario | File | Category |
> |----------|------|----------|
> | Verify new environment variables are set in every deploy target (local, preview, staging, production) — missing env vars are a common source of 500 errors after deploy | `src/config.ts` | regression |
> | Test payment flow end-to-end with test card in a staging environment — verify webhook delivery, idempotency, and correct amount in cents | `src/checkout.ts` | security |
>
> ### Should Test (3)
>
> | Scenario | File | Category |
> |----------|------|----------|
> | Verify retry logic on 5xx from Stripe API | `src/stripe-webhook.ts` | error-handling |
> | Test form submission — valid data, empty fields, special characters, rapid re-submission | `src/components/Form.tsx` | regression |
> | Test date/time logic across timezones and DST boundaries | `src/time.ts` | edge-case |
>
> ---
> *5 scenarios total — PR diff only. [Scan full repo health →](https://qlens.dev?utm_source=qa-advisor&utm_medium=pr-comment&utm_campaign=cta)*

## Usage

Create `.github/workflows/qa-advisor.yml`:

```yaml
name: QA Advisor

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write  # required to post PR comments

jobs:
  advise:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0

      - uses: i-kosheliev/qa-advisor-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          min-priority: nice-to-test  # optional, default: nice-to-test
```

That's it. The action fetches the PR diff, analyzes it, and posts (or updates) a single comment on the PR. Re-runs on each push update the same comment instead of spamming.

Pin to `@v1` for automatic patch + minor updates, or pin to a specific tag (e.g. `@v1.1.0`) for reproducible CI.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `github-token` | yes | — | `secrets.GITHUB_TOKEN` for API access |
| `min-priority` | no | `nice-to-test` | Filter threshold: `must-test`, `should-test`, or `nice-to-test` |

## Outputs

| Output | Description |
|--------|-------------|
| `scenarios-count` | Total scenarios detected |
| `risk-level` | Overall risk: `low`, `medium`, or `high` |
| `summary` | One-line summary of the analysis |

Use outputs in follow-up steps:

```yaml
- uses: i-kosheliev/qa-advisor-action@v1
  id: qa
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}

- name: Fail on high-risk PRs
  if: steps.qa.outputs.risk-level == 'high'
  run: |
    echo "::error::High-risk PR detected. Require senior reviewer sign-off."
    exit 1
```

## What it detects

22 change patterns across 6 categories:

| Category | Example patterns |
|----------|------------------|
| **Security** | auth/login, tokens, CORS, encryption, payments (Stripe/PayPal) |
| **Regression** | input validation, API responses, DB queries, env vars, deps |
| **New behavior** | API endpoints, GraphQL schema, new exports |
| **Edge cases** | conditional rendering, async/Promise, date/time, FS writes |
| **Error handling** | try/catch, throw, `.catch()` |
| **Performance** | loops, iteration, large file changes |

Non-code files (`.github/workflows/*.yml`, lockfiles, `*.md`, `LICENSE`, images) are automatically skipped — no false positives from `secrets.GITHUB_TOKEN` in workflow YAMLs or "auth" in a README.

## Why not an LLM?

- **Instant** — no API latency, runs in seconds on every push.
- **Deterministic** — same diff → same output. Reviewers can trust what they see.
- **Free** — no API costs, no rate limits, no quotas.
- **Private** — your code never leaves GitHub's infrastructure.

For AI-powered test case generation (full test cases with steps, automation code, BDD output), see [CasePilot](https://iklab.dev).

## Related

- **Scan full repo test health** → [qlens.dev](https://qlens.dev) (free for open source)
- **Changelog** → [releases](https://github.com/i-kosheliev/qa-advisor-action/releases)

## License

MIT — built by [IK Lab](https://iklab.dev).
