<!-- markdownlint-disable -->

# Hardening Report: check-spelling--actions-checkout/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--actions-checkout/v4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tag/version strings instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if upstream actions are compromised or tags are moved.

Failing references include:
- check-dist.yml: `actions/checkout@v4.1.6`, `actions/setup-node@v4`, `actions/upload-artifact@v4`
- codeql-analysis.yml: `actions/checkout@v4.1.6`, `github/codeql-action/init@v3`, `github/codeql-action/analyze@v3`
- licensed.yml: `actions/checkout@v4.1.6`
- publish-immutable-actions.yml: `actions/checkout@v4`, `actions/publish-immutable-action@0.0.3`
- test.yml: `actions/setup-node@v4`, `actions/checkout@v4.1.6` (multiple occurrences)
- update-main-version.yml: `actions/checkout@v4.1.6`
- update-test-ubuntu-git.yml: `actions/checkout@v4`, `docker/login-action@v3.3.0`, `docker/build-push-action@v6.5.0`

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:40`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/licensed.yml:11`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:18`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:21`
- `.github/workflows/update-main-version.yml:24`
- `.github/workflows/update-test-ubuntu-git.yml:27`
- `.github/workflows/update-test-ubuntu-git.yml:33`
- `.github/workflows/update-test-ubuntu-git.yml:48`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings, violating sub-rule (a). An attacker who controls the input values can inject arbitrary shell commands.

1. `.github/workflows/update-main-version.yml` line 33: `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}` — both `github.event.inputs.major_version` and `github.event.inputs.target` are workflow_dispatch inputs controlled by the caller and are interpolated directly into the shell command without quoting or env-var indirection.

2. `.github/workflows/update-main-version.yml` line 35: `run: git push origin ${{ github.event.inputs.major_version }} --force` — same issue.

3. `.github/workflows/test.yml` (Verify output step): `echo "Commit: ${{ steps.checkout.outputs.commit }}"`, `echo "Ref: ${{ steps.checkout.outputs.ref }}"`, and `if [ "${{ steps.checkout.outputs.ref }}" != ...` — step outputs are interpolated directly into the shell script.

Locations:

- `.github/workflows/update-main-version.yml:33`
- `.github/workflows/update-main-version.yml:35`
- `.github/workflows/test.yml:253`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

- `check-dist.yml`: job `check-dist` has no permissions block.
- `licensed.yml`: job `test` has no permissions block.
- `test.yml`: jobs `build`, `test`, `test-proxy`, `test-bypass-proxy`, `test-git-container`, and `test-output` all have no permissions blocks.
- `update-main-version.yml`: job `tag` has no permissions block.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 7 workflow files:

**unpinned-uses**: Pinned all action references to full SHA digests:
- actions/checkout@v4.1.6 → @a5ac7e51b41094c92402da3b24376905380afc29
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
- actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- github/codeql-action/init@v3 → @6f5948dfacef28e207b48d0905cf90c03365536d
- github/codeql-action/analyze@v3 → @6f5948dfacef28e207b48d0905cf90c03365536d
- actions/publish-immutable-action@0.0.3 → @4b1aa5c1cde5fedc80d52746c9546cb5560e5f53 (tag was v0.0.3)
- docker/login-action@v3.3.0 → @9780b0c442fbb1117ed29e0efdff1e18412f7567
- docker/build-push-action@v6.5.0 → @5176d81f87c23d6fc96624dfdbcd9f3830bbe445

**script-injection**: Fixed in update-main-version.yml (lines 33, 35) and test.yml (line 253) by moving ${{ }} expressions to env: blocks and referencing them as plain shell variables.

**missing-permissions**: Added permissions blocks to check-dist.yml (contents: read), licensed.yml (contents: read), test.yml (contents: read), and update-main-version.yml (contents: write, needed for git push).

