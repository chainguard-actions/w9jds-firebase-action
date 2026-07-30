<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.25.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.25.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable tag-based references instead of pinned SHA digests, making them vulnerable to supply-chain attacks.

**action.yaml**: `image: docker://w9jds/firebase-action:v15.25.0` — mutable Docker image tag, not a SHA digest.

**build-publish.yaml**: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all tag-based.

**check-release.yml**: `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4` (×2), `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all tag-based.

**docker-build-ci.yml**: `actions/checkout@v4` — tag-based.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:17`
- `.github/workflows/build-publish.yaml:22`
- `.github/workflows/build-publish.yaml:28`
- `.github/workflows/build-publish.yaml:35`
- `.github/workflows/build-publish.yaml:42`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/check-release.yml:25`
- `.github/workflows/check-release.yml:32`
- `.github/workflows/check-release.yml:55`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:70`
- `.github/workflows/check-release.yml:87`
- `.github/workflows/check-release.yml:101`
- `.github/workflows/check-release.yml:109`
- `.github/workflows/check-release.yml:116`
- `.github/workflows/check-release.yml:122`
- `.github/workflows/docker-build-ci.yml:10`

### missing-permissions (severity: medium)

Several workflow files lack a top-level `permissions:` block and have jobs without job-level `permissions:` blocks, meaning those jobs run with the default (potentially broad) token permissions.

**build-publish.yaml**: No top-level permissions; no job-level permissions on the `build` job.

**docker-build-ci.yml**: No top-level permissions; no job-level permissions on the `build` job.

**check-release.yml**: No top-level permissions; the `check-releases` job and the `publish` job have no `permissions:` block (only `bump-version` does).

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

### script-injection (severity: high)

In check-release.yml, two `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, enabling script injection if the values contain shell metacharacters.

**Step 'Remove leading v from version numbers'** (sub-rule a): Directly interpolates `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` into shell variable assignments:
```
FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
```

**Step 'Commit & Push changes'** (sub-rule a): Directly interpolates `${{ needs.check-releases.outputs.firebase-tools-release }}` into git commit message and tag arguments:
```
git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."
```

Locations:

- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:58`
- `.github/workflows/check-release.yml:75`
- `.github/workflows/check-release.yml:76`

### github-env-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' step writes values derived from `needs.check-releases.outputs.*` (untrusted workflow-controlled data) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence the release tag names could inject arbitrary environment variable definitions into subsequent steps.

Offending lines:
```
echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV
```
The shell variables `FIREBASE_ACTIONS_RELEASE` and `FIREBASE_TOOLS_RELEASE` are set directly from `${{ needs.check-releases.outputs.* }}` expressions in the same run block, with no newline sanitization before the write.

Locations:

- `.github/workflows/check-release.yml:59`
- `.github/workflows/check-release.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs with tag comments. Pinned the Docker image in action.yaml to its sha256 digest (preserving the docker:// scheme and tag inline).

2. **missing-permissions**: Added `permissions: {}` top-level blocks to all three workflow files. Added job-level permissions: `contents: read` for checkout-only jobs, `contents: read + packages: write` for Docker push jobs.

3. **script-injection**: In check-release.yml, moved `${{ needs.check-releases.outputs.* }}` expressions from `run:` shell strings into `env:` blocks, then referenced them as plain `$VAR` shell variables in both the 'Remove leading v' step and the 'Commit & Push changes' step.

4. **github-env-injection**: In the 'Remove leading v' step, added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values to `$GITHUB_ENV`, preventing newline injection attacks.

