<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.19.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.19.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable refs instead of pinned SHA digests, making the action vulnerable to supply-chain attacks.

- action.yaml: `image: "docker://w9jds/firebase-action:v15.19.1"` — mutable Docker tag, not a SHA digest.
- build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.
- check-release.yml: `octokit/request-action@v2.x` (×2), `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.
- docker-build-ci.yml: `actions/checkout@v4`.

Locations:

- `action.yaml:15`
- `.github/workflows/build-publish.yaml:18`
- `.github/workflows/build-publish.yaml:23`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:39`
- `.github/workflows/build-publish.yaml:46`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:31`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:67`
- `.github/workflows/check-release.yml:84`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:107`
- `.github/workflows/check-release.yml:114`
- `.github/workflows/check-release.yml:121`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

In check-release.yml, two `run:` blocks directly interpolate `${{ needs.*.outputs.* }}` expressions (sub-rule a) into shell commands, allowing an attacker who can influence the upstream job outputs (e.g. via a crafted release tag name) to inject arbitrary shell commands.

1. 'Remove leading v from version numbers' step (line ~63): directly embeds `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` as bare shell assignments.

2. 'Commit & Push changes' step (line ~78): directly embeds `${{ needs.check-releases.outputs.firebase-tools-release }}` inside `git commit -m`, `git tag -a`, and `git push` shell commands — a crafted tag name could inject shell metacharacters.

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:78`
- `.github/workflows/check-release.yml:79`

### github-env-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' run: step writes values derived from `needs.check-releases.outputs.*` (an untrusted/workflow-controlled source) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The values are first assigned to shell variables via direct `${{ }}` interpolation, then written with `echo "VAR=${VAR#v}" >> $GITHUB_ENV`. A newline embedded in the release tag name could inject arbitrary environment variables into subsequent steps.

Offending lines:
  `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV`
  `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`

Locations:

- `.github/workflows/check-release.yml:65`
- `.github/workflows/check-release.yml:66`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on every job, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- build-publish.yaml: no top-level permissions block; the single `build` job has no permissions block.
- docker-build-ci.yml: no top-level permissions block; the single `build` job has no permissions block.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all action refs to full commit SHAs with tag comments for readability. Pinned the Docker image in action.yaml to its sha256 digest while preserving the `docker://` scheme and tag inline. All 17 locations addressed.

2. **script-injection**: In check-release.yml, moved `${{ needs.check-releases.outputs.* }}` expressions out of `run:` shell strings into the step's `env:` block for both the 'Remove leading v from version numbers' step and the 'Commit & Push changes' step. Shell commands now reference plain environment variables.

3. **github-env-injection**: In check-release.yml, the 'Remove leading v from version numbers' step now sanitizes values with `printf '%s' "..." | tr -d '\n\r'` before writing to `$GITHUB_ENV`, preventing newline injection.

4. **missing-permissions**: Added `permissions: { contents: read, packages: write }` to build-publish.yaml and `permissions: { contents: read }` to docker-build-ci.yml at the top level.

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added explicit `permissions:` blocks to the two jobs that were missing them in `.github/workflows/check-release.yml`: (1) `check-releases` job received `contents: read` (required for GITHUB_TOKEN-authenticated API calls to read release data); (2) `publish` job received `contents: read` (for repo checkout) and `packages: write` (for pushing to GHCR). The `bump-version` job already had `contents: write` and was left unchanged.

