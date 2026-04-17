# QA Advisor — PR Test Coverage Analysis

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-QA_Advisor-blue.svg?logo=github)](https://github.com/marketplace/actions/qa-advisor-pr-test-coverage-analysis)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Node 20](https://img.shields.io/badge/node-%3E=20-brightgreen.svg)](https://nodejs.org/)

**Automatic test scenarios on every PR. Tells reviewers exactly what to verify.**

QA Advisor analyzes your PR diff and posts a single comment listing test scenarios the reviewer should check — ranked `must-test`, `should-test`, `nice-to-test`. Detects 17 change patterns across 6 categories: security, regression, new behavior, edge cases, error handling, performance.

**No LLM. No API keys. No external services.** Local heuristic — same diff produces the same output, every time.

---

## Example PR comment

> ## QA Advisor: Test Coverage Analysis
>
> **Risk Level: MEDIUM** 🟡 | 3 file(s) changed (+142/-28 lines)
>
> ### Must Test (2)
>
> | Scenario | File | Category |
> |----------|------|----------|
> | Verify webhook signature validation rejects tampered payloads | `src/stripe-webhook.ts` | security |
> | Ensure subscription downgrade preserves historical data | `src/subscription.ts` | regression |
>
> ### Should Test (3)
>
> | Scenario | File | Category |
> |----------|------|----------|
> | Verify retry logic on 5xx from Stripe API | `src/stripe-webhook.ts` | error-handling |
> | ... | | |
>
> ---
> *5 scenario(s) total — PR diff only. [Scan full repo health →](https://qlens.dev)*
> *Powered by QA Advisor from IK Lab*

---

## Quick start

Create `.github/workflows/qa-advisor.yml`:

```yaml
name: QA Advisor

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  advise:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write  # required to post PR comments
      contents: read
    steps:
      - uses: i-kosheliev/qa-advisor-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          min-priority: should-test  # optional, default: nice-to-test
```

That's it. The action fetches the PR diff, analyzes it, and posts (or updates) a single comment. Re-runs on each push update the same comment instead of spamming.

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

### Use outputs for gating

Fail the build on high-risk PRs, or require senior-review approval:

```yaml
- uses: i-kosheliev/qa-advisor-action@v1
  id: qa
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}

- name: Fail on high-risk PRs
  if: steps.qa.outputs.risk-level == 'high'
  run: |
    echo "::error::High-risk PR. Require senior reviewer sign-off."
    exit 1
```

## What it detects

17 change patterns across 6 categories:

| Category | Examples |
|----------|----------|
| **Security** | Auth logic changes, token handling, CORS, SQL injection vectors, secret exposure |
| **Regression** | Modified existing behavior, deleted branches, changed return types |
| **New behavior** | Added exports, new endpoints, new conditionals |
| **Edge cases** | Array boundaries, null/undefined handling, empty-string checks |
| **Error handling** | New try/catch blocks, error propagation, fallback logic |
| **Performance** | Loops over arrays, database queries, file I/O |

Each detected pattern produces a targeted scenario with the exact file that triggered it. No generic "test everything" advice.

## Why not an LLM?

- **Instant.** No API latency — runs in seconds on every push.
- **Deterministic.** Same diff → same output. Reviewers trust what they see.
- **Free.** No API costs, no rate limits, no quotas.
- **Private.** Your code never leaves GitHub's infrastructure. No third-party API call.
- **CI-friendly.** No secrets to manage, no key rotation.

## How does it compare?

| Feature | QA Advisor | Copilot PR review | Codecov | BuildPulse |
|---------|-----------|-------------------|---------|-----------|
| PR test scenarios | Yes | Partial | No | No |
| Deterministic output | Yes | No (LLM) | Yes | Yes |
| No external API | Yes | No | No | No |
| Free for public repos | Yes | Paid | Free tier | Paid |
| Setup friction | One workflow file | Enterprise | CI + coverage files | CI integration |
| Runs on every push | Yes | Yes | Requires coverage | Requires test runs |

QA Advisor is complementary to code coverage and flaky-test detection — it tells reviewers **what to test**, not whether existing tests pass.

## Want deeper repo-level analysis?

This action scans **only the PR diff**. For full-repository test health — A–F grade, structural flakiness detection, historical trends — scan your repo on **[qlens.dev](https://qlens.dev)**.

Free tier: one scan per repo. Sign in with GitHub, no credit card.

## License

MIT — see [LICENSE](LICENSE). Built by [IK Lab](https://iklab.dev).

## Issues & feature requests

Open an issue: https://github.com/i-kosheliev/qa-advisor-action/issues
