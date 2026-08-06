<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.26.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.26.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml uses a mutable Docker image tag instead of a SHA digest: `image: docker://w9jds/firebase-action:v15.26.0`. This is vulnerable to supply-chain attacks if the tag is moved. All three workflow files also use mutable action refs instead of pinned 40-character SHA commits:
- build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`
- check-release.yml: `octokit/request-action@v2.x` (×2), `madhead/semver-utils@latest`, `actions/checkout@v4` (×2), `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`
- docker-build-ci.yml: `actions/checkout@v4`

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:16`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/docker-build-ci.yml:9`

### permissions (severity: medium)

Missing `permissions:` declarations:
- `build-publish.yaml`: No top-level `permissions:` key and no per-job `permissions:` key on any job. The workflow runs on `pull_request` events and pushes to master/tags, but has no permission restrictions.
- `docker-build-ci.yml`: No top-level `permissions:` key and no per-job `permissions:` key. The workflow runs on `pull_request` and `workflow_dispatch` with no permission restrictions.
- `check-release.yml`: No top-level `permissions:` key. Only the `bump-version` job has a `permissions:` block (`contents: write`). The `check-releases` and `publish` jobs have no `permissions:` key, so they inherit the default broad permissions.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

### script-injection (severity: high)

Direct `${{ ... }}` expression interpolation inside `run:` shell commands in check-release.yml:

(a) "Remove leading 'v' from version numbers" step (around line 57): `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments. An attacker who can influence a release tag name could inject shell metacharacters.

(a) "Commit & Push changes" step (around line 79): `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into `git commit -a -m "..."` and `git tag -a ${{ ... }} -m "..."` shell commands. A malicious release tag name could inject arbitrary shell commands.

Locations:

- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:79`

### github-env-injection (severity: high)

In check-release.yml, the "Remove leading 'v' from version numbers" step writes values derived from `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` directly to `$GITHUB_ENV` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). These values come from `needs.*.outputs.*` which is an untrusted-input source. A malicious release tag containing newlines could inject arbitrary environment variable definitions into subsequent steps:
```
FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV
```

Locations:

- `.github/workflows/check-release.yml:60`
- `.github/workflows/check-release.yml:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all action refs to full 40-char SHAs with tag comments. Pinned the Docker image in action.yaml to its SHA digest while preserving the docker:// scheme and tag.

2. **permissions**: Added `permissions: {}` at top level of all three workflow files. Added minimal per-job permissions: `contents: read` for checkout-only jobs, `contents: read, packages: write` for Docker push jobs, and `contents: write` was already present on bump-version.

3. **script-injection**: Moved all `${{ needs.check-releases.outputs.* }}` expressions out of `run:` shell commands into `env:` blocks. Shell scripts now reference plain environment variables.

4. **github-env-injection**: Values written to $GITHUB_ENV are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection attacks.

