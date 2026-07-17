<!-- markdownlint-disable -->

# Hardening Report: actions--deploy-pages/v4.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--deploy-pages/v4.0.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.ref_name }}` is interpolated directly inside a `run:` shell command string in two places. This allows an attacker who controls the branch name (e.g. via a crafted Dependabot PR branch) to inject arbitrary shell commands. Offending lines:
  - `echo "Pushing branch ${{ github.ref_name }}"`
  - `git push origin ${{ github.ref_name }}`
Fix: move the value into an `env:` variable and double-quote it in the shell: `env: REF_NAME: ${{ github.ref_name }}` then `git push origin "$REF_NAME"`.

Locations:

- `.github/workflows/rebuild-dependabot-prs.yml:46`
- `.github/workflows/rebuild-dependabot-prs.yml:47`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags (e.g. `@v4`, `@v3`, `@v0.3.0`) instead of immutable 40-character commit SHAs. This exposes the workflows to supply-chain attacks if the upstream action tag is moved to malicious code.

Failing references:
- check-dist.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`
- check-formatting.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- check-linter.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- rebuild-dependabot-prs.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- release.yml: `actions/publish-action@v0.3.0`
- test.yml: `actions/checkout@v4`, `actions/setup-node@v4`

Note: `draft-release.yml` correctly pins `release-drafter/release-drafter` to a full SHA and is not a finding.

Locations:

- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/check-formatting.yml:21`
- `.github/workflows/check-formatting.yml:24`
- `.github/workflows/check-linter.yml:21`
- `.github/workflows/check-linter.yml:24`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/rebuild-dependabot-prs.yml:23`
- `.github/workflows/rebuild-dependabot-prs.yml:27`
- `.github/workflows/release.yml:22`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:21`

### missing-permissions (severity: medium)

`check-dist.yml` has no top-level `permissions:` key and no job-level `permissions:` key on the `check-dist` job. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be `write-all` depending on repository settings. All other workflow files in this repository define explicit permissions.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 7 workflow files:

1. script-injection: In rebuild-dependabot-prs.yml, moved `${{ github.ref_name }}` from the run: shell string into an env: block as REF_NAME. The shell now uses `echo "Pushing branch $REF_NAME"` and `git push origin "$REF_NAME"` (double-quoted) to prevent shell injection.

2. unpinned-uses: Pinned all 16 unpinned action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
   - github/codeql-action/{init,autobuild,analyze}@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
   - actions/publish-action@v0.3.0 → @f784495ce78a41bac4ed7e34a73f0034015764bb

3. missing-permissions: Added `permissions: contents: read` top-level block to check-dist.yml (the only workflow file that was missing an explicit permissions declaration).

