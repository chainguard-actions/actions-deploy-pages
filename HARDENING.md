<!-- markdownlint-disable -->

# Hardening Report: actions--deploy-pages/v5.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--deploy-pages/v5.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved or the action is compromised. Unpinned references found:
- check-dist.yml: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4
- check-formatting.yml: actions/checkout@v4, actions/setup-node@v4
- check-linter.yml: actions/checkout@v4, actions/setup-node@v4
- codeql-analysis.yml: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3
- publish-immutable-actions.yml: actions/checkout@v4, actions/publish-immutable-action@0.0.3
- rebuild-dependabot-prs.yml: actions/checkout@v4, actions/setup-node@v4
- release.yml: actions/publish-action@v0.3.0
- test.yml: actions/checkout@v4, actions/setup-node@v4

Locations:

- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/check-formatting.yml:21`
- `.github/workflows/check-formatting.yml:24`
- `.github/workflows/check-linter.yml:21`
- `.github/workflows/check-linter.yml:24`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/codeql-analysis.yml:56`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/rebuild-dependabot-prs.yml:24`
- `.github/workflows/rebuild-dependabot-prs.yml:29`
- `.github/workflows/release.yml:22`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:19`

### missing-permissions (severity: medium)

check-dist.yml has no top-level `permissions:` key and its only job (`check-dist`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default GITHUB_TOKEN permissions, which may be broader than necessary.

Locations:

- `.github/workflows/check-dist.yml:1`

### script-injection (severity: high)

Sub-rule (a): In rebuild-dependabot-prs.yml, the `run:` block directly interpolates `${{ github.ref_name }}` into a shell command: `git push origin ${{ github.ref_name }}`. The GitHub Actions expression is substituted into the shell command string before the shell executes it, allowing an attacker who controls the branch name to inject arbitrary shell commands (e.g., a branch named `main; malicious-command`). The value should be passed via an `env:` variable and double-quoted in the shell instead.

Locations:

- `.github/workflows/rebuild-dependabot-prs.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments in check-dist.yml, check-formatting.yml, check-linter.yml, codeql-analysis.yml, publish-immutable-actions.yml, rebuild-dependabot-prs.yml, release.yml, and test.yml.

2. missing-permissions: Added 'permissions: contents: read' top-level block to check-dist.yml (the only workflow missing it).

3. script-injection: In rebuild-dependabot-prs.yml, moved ${{ github.ref_name }} to an env: block as REF_NAME and replaced the unquoted shell interpolation with double-quoted "$REF_NAME".

Note: actions/publish-immutable-action was referenced as @0.0.3 (no v prefix) in the original but the actual tag is v0.0.3; the SHA from v0.0.3 was used and the comment reflects the original tag format.

