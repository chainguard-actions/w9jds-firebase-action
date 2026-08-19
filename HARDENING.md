<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.22.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a mutable Docker image tag instead of a SHA digest: `image: "docker://w9jds/firebase-action:v15.22.0"`. This is vulnerable to supply-chain attacks because the tag can be silently overwritten to point to a different image.

Locations:

- `action.yaml:16`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags rather than full 40-character commit SHAs. Affected references: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2`, `docker/build-push-action@v3` (build-publish.yaml); `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2`, `docker/build-push-action@v3` (check-release.yml); `actions/checkout@v4` (docker-build-ci.yml).

Locations:

- `.github/workflows/build-publish.yaml:18`
- `.github/workflows/build-publish.yaml:24`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:38`
- `.github/workflows/build-publish.yaml:45`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:35`
- `.github/workflows/check-release.yml:56`
- `.github/workflows/check-release.yml:72`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:112`
- `.github/workflows/check-release.yml:119`
- `.github/workflows/check-release.yml:126`
- `.github/workflows/check-release.yml:133`
- `.github/workflows/check-release.yml:140`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside `run:` blocks. In the 'Remove leading v from version numbers' step, `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments — an attacker who controls a release tag name could inject arbitrary shell commands. In the 'Commit & Push changes' step, `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into `git commit -m` and `git tag -a` arguments. Offending lines:
  `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}`
  `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}`
  `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."`

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:88`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' step writes values derived from `needs.check-releases.outputs.*` (which originate from external GitHub API responses for release tag names) to `$GITHUB_ENV` without sanitization. The values are first assigned to shell variables via direct expression interpolation (`${{ needs.check-releases.outputs.firebase-actions-release }}`), then written unsanitized with `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV` and `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`. A malicious release tag containing newlines could inject arbitrary environment variables. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/check-release.yml:65`
- `.github/workflows/check-release.yml:66`

### missing-permissions (severity: medium)

Workflow files `build-publish.yaml` and `docker-build-ci.yml` have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the default token permissions (which may be broad, depending on repository settings) are granted to the GITHUB_TOKEN, violating the principle of least privilege.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 4 files:

1. action.yaml: Pinned docker://w9jds/firebase-action:v15.22.0 to its SHA digest (sha256:f9479aecc3ef32e5c699c812283f709801e9196453db8a79905772bb06ba30d8), preserving the docker:// scheme and tag inline.

2. build-publish.yaml: Pinned all 5 action references to full commit SHAs (actions/checkout, docker/metadata-action, docker/login-action x2, docker/build-push-action). Added top-level permissions block (contents: read, packages: write).

3. docker-build-ci.yml: Pinned actions/checkout to full commit SHA. Added top-level permissions block (contents: read).

4. check-release.yml: Pinned all 8 action references to full commit SHAs. Fixed script-injection by moving ${{ needs.check-releases.outputs.* }} expressions into env: blocks and referencing them as plain shell variables. Fixed github-env-injection by sanitizing values with printf | tr -d '\n\r' before writing to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added explicit permissions to check-release.yml: (1) top-level `permissions: contents: read` as a safe restrictive default; (2) job-level `permissions: contents: read` to the `check-releases` job; (3) job-level `permissions: contents: read` and `packages: write` to the `publish` job (packages: write is needed to push to GHCR). The `bump-version` job already had `permissions: contents: write` and was left unchanged.

