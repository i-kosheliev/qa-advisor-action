# Changelog — QA Advisor

Notable changes. This is a distribution-only repo — source lives in the [casepilot monorepo](https://github.com/i-kosheliev/casepilot/tree/master/packages/github-action-qa-advisor) and is synced here on release.

## v1.2.0 — 2026-04-18

### Added

- **3 new change patterns** (25 total, was 22):
  - `cache-invalidation` (must-test, regression) — `cache.set/delete/clear`, `revalidatePath/revalidateTag`, `unstable_cache`, `cacheLife/cacheTag/updateTag`, `redis.set/del`, `memcache.*`. Stale cache is silent: reads look fine, writes don't propagate until TTL.
  - `feature-flag` (should-test, edge-case) — `featureFlag`, `isEnabled`, `growthbook`, `launchDarkly`, `flagOn`, `getVariant`, `getBooleanValue`, `experiment`, `variation`, `unleash.*`, `posthog.isFeatureEnabled`. Nudges reviewer to test BOTH branches + the fallback when the flag service is unreachable.
  - `dangerous-sink` (must-test, security) — `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `document.write`, `eval`, `new Function`, `setAttribute('srcdoc'`, `insertAdjacentHTML`. XSS / arbitrary code execution surface.

### Tests

- 6 new pr-test-advisor tests (one per new pattern + nonCode regression + secondary-trigger samples)
- 75/75 test suite pass

---

## v1.1.0 — 2026-04-17

### Added

- **5 new change patterns** (22 total, was 17):
  - `env-var-usage` (must-test) — `process.env.X`, `Deno.env.get`, `import.meta.env.X`, `os.environ`, `os.getenv`. Catches new env var references that need provisioning in every deploy target.
  - `payment-integration` (must-test, security) — `stripe.*`, `paymentIntent`, `checkout.sessions`, `subscriptions.*`, `paypal.*`, `braintree.*`.
  - `fs-write` (should-test, edge-case) — `fs.writeFile`, `fs.writeFileSync`, `fs.unlink`, `fs.mkdir`, `fs.createWriteStream`. Serverless read-only FS footgun.
  - `graphql-schema` (must-test, new-behavior) — `gql\`\``, `typeDefs`, `buildSchema`, `makeExecutableSchema`, paths `.graphql` / `.gql` / `schema.ts` / `resolvers.ts`.
  - `datetime-logic` (should-test, edge-case) — `new Date()`, `Date.now()`, `dayjs()`, `moment()`, `toLocaleDate`, `toISOString`, `getTimezoneOffset`, `setHours`, `addDays`, `formatDistance`.
- **UTM-tagged CTA** in footer — `utm_source=qa-advisor&utm_medium=pr-comment&utm_campaign=cta` / `…campaign=footer`. Lets QualityPilot attribute PR-comment clicks back to this action.
- Marketplace badges + a 6-category pattern table in README.

### Fixed

- **Pluralization** — `"1 file(s) changed"` / `"1 scenario(s) total"` → `"1 file changed"` / `"1 scenario total"` / `"3 files changed"` / `"3 scenarios total"`.

### Compatibility

- All 22 patterns respect `isNonCodeFile()` from v1.0.1 — workflow YAMLs mentioning `process.env` or `stripe.` still produce 0 scenarios.

### Tests

- 7 new `pr-test-advisor` tests (one per new pattern + nonCode regression)
- 3 new `formatter` tests (singular/plural file + scenario + UTM)
- 1020/1020 monorepo + 15/15 action tests pass

---

## v1.0.2 — 2026-04-17

### Changed

- **Node.js runtime** `node20 → node24`. Ahead of GitHub's Node 20 deprecation (forced to Node 24 on 2026-06-02, removed 2026-09-16). `action.yml` `using:` and `esbuild` target both bumped. Bundle bytes unchanged — no code uses features that differ between those targets.

---

## v1.0.1 — 2026-04-17

### Fixed

- **False positives on non-code files.** Pre-fix, a PR that only added `.github/workflows/qa-advisor.yml` was flagged MEDIUM with 2 must-test security scenarios because the workflow contained `secrets.GITHUB_TOKEN` — substring `"token"` matched the `auth-change` linePattern, `"secret"` matched `security-change`.

New `isNonCodeFile()` filter skips line-pattern matching on:

- `.github/workflows/*.y?ml`
- Lockfiles: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `composer.lock`, `Gemfile.lock`, `poetry.lock`, `cargo.lock`, `go.sum`
- Docs: `*.md`, `*.mdx`, `*.txt`, `*.rst`, `*.adoc`
- Metadata: `LICENSE`, `NOTICE`, `AUTHORS`, `CONTRIBUTORS`, `CHANGELOG`, `README`, `CODEOWNERS`
- Dot-files: `.gitignore`, `.gitattributes`, `.editorconfig`, etc.
- Images: `*.png`, `*.jpg`, `*.gif`, `*.svg`, `*.ico`, `*.webp`, `*.bmp`, `*.tiff`

Path-pattern rules still fire (`dependency-change` still detects `package-lock.json`).

---

## v1.0.0 — 2026-04-17

Initial release. 17 change patterns, local heuristic, posts test scenarios as a PR comment. Published to GitHub Marketplace: [QA Advisor — PR Test Coverage Analysis](https://github.com/marketplace/actions/qa-advisor-pr-test-coverage-analysis).
