<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.25.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.25.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml uses a mutable Docker image tag instead of a SHA digest: `image: docker://w9jds/firebase-action:v15.25.1`. This is vulnerable to supply-chain attacks if the tag is moved. All three workflow files also use tag-based (non-SHA-pinned) `uses:` references:
- build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3
- check-release.yml: octokit/request-action@v2.x (×2), madhead/semver-utils@latest, actions/checkout@v4, jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3
- docker-build-ci.yml: actions/checkout@v4

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:17`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `needs.*.outputs.*` values inside `run:` shell commands. In the 'Remove leading v from version numbers' step, `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments — an attacker who can influence a GitHub release tag name could inject arbitrary shell commands. In the 'Commit & Push changes' step, `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into `git commit -a -m` and `git tag -a` commands.

Locations:

- `.github/workflows/check-release.yml:61`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:80`
- `.github/workflows/check-release.yml:81`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' run: step writes values derived from `needs.check-releases.outputs.firebase-actions-release` and `needs.check-releases.outputs.firebase-tools-release` (untrusted `needs.*.outputs.*` values) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in a release tag name could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`

### missing-permissions (severity: medium)

build-publish.yaml has no top-level `permissions:` key and no job-level `permissions:` key on any job, granting the default (broad) token permissions to all steps. docker-build-ci.yml similarly has no top-level or job-level `permissions:` key. check-release.yml has a `permissions:` key only on the `bump-version` job; the `check-releases` and `publish` jobs have no `permissions:` key and there is no top-level `permissions:` key, so those jobs run with default broad permissions.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across four files:

1. **action.yaml**: Pinned Docker image `w9jds/firebase-action:v15.25.1` to its SHA digest (`sha256:a19e6a17a4f3699e33a61a649a2afec343ae75531f7dfb4dc04f2888a088fd81`), preserving the `docker://` scheme.

2. **build-publish.yaml**: Pinned all 5 action `uses:` references to full commit SHAs (with tag comments). Added top-level `permissions: { contents: read, packages: write }`.

3. **docker-build-ci.yml**: Pinned `actions/checkout@v4` to full commit SHA. Added top-level `permissions: { contents: read }`.

4. **check-release.yml**: 
   - Pinned all 9 action `uses:` references to full commit SHAs.
   - Added top-level `permissions: { contents: read }`, job-level `permissions: { contents: read }` on `check-releases` job, and `permissions: { contents: read, packages: write }` on `publish` job.
   - Fixed script injection: moved `needs.check-releases.outputs.*` expressions into `env:` blocks in both the 'Remove leading v' and 'Commit & Push changes' steps.
   - Fixed github-env-injection: sanitized values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

