<!-- markdownlint-disable -->

# Hardening Report: actions--deploy-pages/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--deploy-pages/v5.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised.

Failing references:
- check-dist.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`
- check-formatting.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- check-linter.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- publish-immutable-actions.yml: `actions/checkout@v4`, `actions/publish-immutable-action@0.0.3`
- rebuild-dependabot-prs.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- release.yml: `actions/publish-action@v0.3.0`
- test.yml: `actions/checkout@v4`, `actions/setup-node@v4`

Locations:

- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:26`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/check-formatting.yml:20`
- `.github/workflows/check-formatting.yml:24`
- `.github/workflows/check-linter.yml:20`
- `.github/workflows/check-linter.yml:24`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:50`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/rebuild-dependabot-prs.yml:24`
- `.github/workflows/rebuild-dependabot-prs.yml:29`
- `.github/workflows/release.yml:24`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:20`

### missing-permissions (severity: medium)

The workflow file `check-dist.yml` has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the workflow runs with the default GITHUB_TOKEN permissions, which may be broader than necessary (e.g., write access to contents on some repository configurations).

Locations:

- `.github/workflows/check-dist.yml:1`

### script-injection (severity: high)

Sub-rule (a): The `rebuild-dependabot-prs.yml` workflow directly interpolates the GitHub Actions expression `${{ github.ref_name }}` inside a `run:` shell script block. Before the shell executes the script, GitHub Actions performs template substitution, replacing the expression with the raw value of `github.ref_name`. An attacker who can control the branch name (e.g., by creating a branch with shell metacharacters) could inject arbitrary shell commands.

Offending lines:
  `echo "Pushing branch ${{ github.ref_name }}"`
  `git push origin ${{ github.ref_name }}`

Fix: Move the value into an env var and quote it: `env: REF_NAME: ${{ github.ref_name }}` then use `"$REF_NAME"` in the run block.

Locations:

- `.github/workflows/rebuild-dependabot-prs.yml:47`
- `.github/workflows/rebuild-dependabot-prs.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 18 unpinned action references across check-dist.yml, check-formatting.yml, check-linter.yml, codeql-analysis.yml, publish-immutable-actions.yml, rebuild-dependabot-prs.yml, release.yml, and test.yml by pinning each to its full commit SHA with the original tag as a comment. Added a top-level `permissions: contents: read` block to check-dist.yml to address the missing-permissions finding. Fixed the script injection in rebuild-dependabot-prs.yml by moving `${{ github.ref_name }}` into a step-level `env:` block as `REF_NAME` and referencing it as `"$REF_NAME"` in the shell script.

