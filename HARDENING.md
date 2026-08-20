<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.28.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.28.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references and the Docker image in action.yaml are pinned to mutable tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks.

action.yaml: `image: docker://w9jds/firebase-action:v15.28.1` (mutable tag, not a SHA digest).

build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

check-release.yml: `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

docker-build-ci.yml: `actions/checkout@v4`.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:18`
- `.github/workflows/build-publish.yaml:23`
- `.github/workflows/build-publish.yaml:29`
- `.github/workflows/build-publish.yaml:35`
- `.github/workflows/build-publish.yaml:41`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:32`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:70`
- `.github/workflows/check-release.yml:88`
- `.github/workflows/check-release.yml:107`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:119`
- `.github/workflows/check-release.yml:125`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

Two `run:` blocks in check-release.yml directly interpolate `${{ ... }}` expressions into shell commands, violating rule (a).

(1) 'Remove leading v from version numbers' step (line 63): `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments. An attacker who controls a release tag name could inject arbitrary shell commands.

(2) 'Commit & Push changes' step (line 82): `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into `git commit -m` and `git tag -a` shell arguments. A malicious release tag name could break out of the quoted string and execute arbitrary commands.

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:82`
- `.github/workflows/check-release.yml:83`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' run block in check-release.yml writes values derived from `needs.check-releases.outputs.*` (which originate from external GitHub API responses for release tag names) into `$GITHUB_ENV` without sanitization. The values are first interpolated directly via `${{ ... }}` into shell variables (itself a script-injection risk), and then written to GITHUB_ENV via `echo "VAR=${VAR#v}" >> $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, so a release tag name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/check-release.yml:65`
- `.github/workflows/check-release.yml:66`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` block and no job-level `permissions:` on any of their jobs, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- build-publish.yaml: no top-level permissions, and neither the `build` job nor any other job defines a `permissions:` key.
- docker-build-ci.yml: no top-level permissions and the single `build` job has no `permissions:` key.

check-release.yml also lacks top-level permissions and the `check-releases` and `publish` jobs have no job-level permissions (only `bump-version` does), so those jobs also run with default broad permissions.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-character commit SHAs with tag comments for readability. Pinned the Docker image in action.yaml to its sha256 digest while preserving the `docker://` scheme and tag inline.

2. **script-injection**: In check-release.yml, moved `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` out of `run:` blocks into `env:` blocks, then referenced them as `$FIREBASE_ACTIONS_RELEASE_RAW`, `$FIREBASE_TOOLS_RELEASE_RAW`, and `$FIREBASE_TOOLS_RELEASE` in the shell scripts.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to `$GITHUB_ENV` in the 'Remove leading v' step, and similarly sanitized the value used in git commit/tag messages.

4. **missing-permissions**: Added top-level `permissions: contents: read` to all three workflow files (build-publish.yaml, docker-build-ci.yml, check-release.yml). Added job-level permissions to check-releases (contents: read) and publish (contents: read, packages: write) jobs in check-release.yml. build-publish.yaml got top-level `contents: read, packages: write`.

