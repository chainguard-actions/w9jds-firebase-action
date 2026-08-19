<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.20.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags are moved. Unpinned references found:

build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

check-release.yml: octokit/request-action@v2.x (×2), madhead/semver-utils@latest, actions/checkout@v4 (×2), jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

docker-build-ci.yml: actions/checkout@v4

action.yaml: The `runs.image` field references `docker://w9jds/firebase-action:v15.20.0` — a mutable Docker image tag rather than a SHA digest.

Locations:

- `.github/workflows/build-publish.yaml:19`
- `.github/workflows/build-publish.yaml:24`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:38`
- `.github/workflows/build-publish.yaml:44`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:32`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:70`
- `.github/workflows/check-release.yml:82`
- `.github/workflows/check-release.yml:93`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:107`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/docker-build-ci.yml:9`
- `action.yaml:15`

### script-injection (severity: high)

Two `run:` blocks in check-release.yml directly interpolate `${{ needs.*.outputs.* }}` expressions (workflow-controllable context) into shell command strings, violating rule (a).

1. 'Remove leading v from version numbers' step (line ~62): `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}` and `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}` are substituted directly into the shell script before execution. An attacker who can influence the release tag name could inject arbitrary shell commands.

2. 'Commit & Push changes' step (line ~75): `git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"` and `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."` interpolate the output value directly into git commands without quoting or sanitization.

Locations:

- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:75`
- `.github/workflows/check-release.yml:76`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' step in check-release.yml writes values derived from `needs.check-releases.outputs.*` (workflow-controllable context) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The values are first assigned to shell variables via direct `${{ }}` interpolation and then written unsanitized:

  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

A newline embedded in a release tag name could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:65`

### missing-permissions (severity: medium)

Several workflow files lack `permissions:` declarations, meaning jobs run with the default (potentially broad) token permissions:

- build-publish.yaml: No top-level `permissions:` key and the `build` job has no job-level `permissions:` key.
- docker-build-ci.yml: No top-level `permissions:` key and the `build` job has no job-level `permissions:` key.
- check-release.yml: No top-level `permissions:` key; the `check-releases` job and the `publish` job both lack job-level `permissions:` keys (only `bump-version` has `permissions: contents: write`).

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:9`
- `.github/workflows/check-release.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across four files:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-char commit SHAs with tag comments: actions/checkout@v4→11d5960a, docker/metadata-action@v4→818d4b7b, docker/login-action@v2→465a0781, docker/build-push-action@v3→1104d471, octokit/request-action@v2.x→02f5e7c6, madhead/semver-utils@latest→4cf918af, jacobtomlinson/gha-find-replace@v3→2ff30f64, comnoco/create-release-action@v2.0.5→7dea6dc8. Pinned action.yaml Docker image to sha256:06a24126... while preserving docker:// scheme and :v15.20.0 tag inline.

2. **script-injection**: Moved `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` out of `run:` blocks into `env:` blocks; referenced as plain shell variables in the scripts.

3. **github-env-injection**: Added `printf '%s' "..." | tr -d '\n\r'` sanitization before writing stripped version values to $GITHUB_ENV.

4. **missing-permissions**: Added `permissions: contents: read` top-level to build-publish.yaml and docker-build-ci.yml (with packages: write for build-publish.yaml). Added top-level and per-job permissions to check-release.yml for check-releases (contents: read) and publish (contents: read, packages: write) jobs.

