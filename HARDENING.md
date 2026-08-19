<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.16.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.16.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'Remove leading v from version numbers' step, `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into the shell script before the shell ever sees them. An attacker who can influence the upstream firebase-tools release tag name could inject arbitrary shell commands. Offending lines:
  `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}`
  `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}`

Locations:

- `.github/workflows/check-release.yml:63`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'Commit & Push changes' step, `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into git commit message and git tag arguments in the shell script. An attacker who can influence the upstream release tag name could inject arbitrary shell commands. Offending lines:
  `git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"`
  `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"`

Locations:

- `.github/workflows/check-release.yml:80`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' step writes values derived from `needs.check-releases.outputs.firebase-actions-release` and `needs.check-releases.outputs.firebase-tools-release` (which originate from external GitHub API responses for release tag names) to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in a release tag name could inject arbitrary environment variables into subsequent steps. Offending lines:
  `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV`
  `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`

Locations:

- `.github/workflows/check-release.yml:65`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the branch is updated.

build-publish.yaml:
  - `actions/checkout@v4`
  - `docker/metadata-action@v4`
  - `docker/login-action@v2` (×2)
  - `docker/build-push-action@v3`

check-release.yml:
  - `octokit/request-action@v2.x` (×2)
  - `madhead/semver-utils@latest`
  - `actions/checkout@v4`
  - `jacobtomlinson/gha-find-replace@v3`
  - `comnoco/create-release-action@v2.0.5`
  - `docker/metadata-action@v4`
  - `docker/login-action@v2` (×2)
  - `docker/build-push-action@v3`

docker-build-ci.yml:
  - `actions/checkout@v4`

Locations:

- `.github/workflows/build-publish.yaml:18`
- `.github/workflows/build-publish.yaml:24`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:39`
- `.github/workflows/build-publish.yaml:47`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:31`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:70`
- `.github/workflows/check-release.yml:88`
- `.github/workflows/check-release.yml:107`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:119`
- `.github/workflows/check-release.yml:125`
- `.github/workflows/docker-build-ci.yml:10`

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag (`v15.16.0`) instead of a SHA digest. If the image at this tag is replaced or overwritten, the action will silently execute different code. The `image: docker://w9jds/firebase-action:v15.16.0` field should use a SHA digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:16`

### permissions (severity: medium)

missing-permissions: The workflow file has no top-level `permissions:` key and the jobs within it have no job-level `permissions:` blocks either. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be read/write), violating the principle of least privilege.

Locations:

- `.github/workflows/build-publish.yaml:1`

### permissions (severity: medium)

missing-permissions: The workflow file has no top-level `permissions:` key and its single job has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be read/write), violating the principle of least privilege.

Locations:

- `.github/workflows/docker-build-ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, permissions

**Notes:**

Fixed all findings across 4 files:

1. action.yaml: Pinned Docker image `w9jds/firebase-action:v15.16.0` to SHA digest `sha256:bbd3e753c68a922db946027dce342491753f1b33b71cb5321dfd4869acbcfbe3` while preserving the `docker://` scheme and tag.

2. build-publish.yaml: Pinned all 5 action references to full SHAs (actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 ×2, docker/build-push-action@v3). Added top-level `permissions: contents: read, packages: write`.

3. docker-build-ci.yml: Pinned actions/checkout@v4 to full SHA. Added top-level `permissions: contents: read`.

4. check-release.yml: (a) Pinned all 8 action references to full SHAs. (b) Fixed script-injection in 'Remove leading v from version numbers' step by moving expressions to `env:` block and referencing plain env vars in the shell. (c) Fixed script-injection in 'Commit & Push changes' step by moving the release expression to `env:` block. (d) Fixed github-env-injection by sanitizing values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added a top-level `permissions: {}` block to deny all permissions by default. Added `permissions: contents: read` to the `check-releases` job (needs read access for GitHub API release lookups) and to the `publish` job (needs read access for checkout; Docker/GHCR logins use external secrets). The `bump-version` job already had `permissions: contents: write` and was left unchanged.

