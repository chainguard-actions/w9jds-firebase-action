<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.22.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml reference actions and Docker images by mutable tags/branches instead of pinned full SHA digests, making the action vulnerable to supply-chain attacks.

.github/workflows/build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

.github/workflows/check-release.yml: octokit/request-action@v2.x (×2), madhead/semver-utils@latest, actions/checkout@v4, jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

.github/workflows/docker-build-ci.yml: actions/checkout@v4

action.yaml: runs.image is 'docker://w9jds/firebase-action:v15.22.4' — a mutable tag, not a SHA digest.

Locations:

- `.github/workflows/build-publish.yaml:20`
- `.github/workflows/build-publish.yaml:25`
- `.github/workflows/build-publish.yaml:33`
- `.github/workflows/build-publish.yaml:40`
- `.github/workflows/build-publish.yaml:47`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:31`
- `.github/workflows/check-release.yml:55`
- `.github/workflows/check-release.yml:67`
- `.github/workflows/check-release.yml:84`
- `.github/workflows/check-release.yml:97`
- `.github/workflows/check-release.yml:105`
- `.github/workflows/check-release.yml:112`
- `.github/workflows/check-release.yml:119`
- `.github/workflows/check-release.yml:124`
- `.github/workflows/docker-build-ci.yml:11`
- `action.yaml:15`

### permissions (severity: medium)

missing-permissions: These workflow files have no top-level 'permissions:' key and at least one job also lacks a job-level 'permissions:' key, meaning jobs run with the default (potentially broad) token permissions.

- build-publish.yaml: No top-level permissions and the single 'build' job has no permissions block.
- docker-build-ci.yml: No top-level permissions and the single 'build' job has no permissions block.
- check-release.yml: No top-level permissions; the 'check-releases' job and 'publish' job have no permissions block (only 'bump-version' has 'permissions: contents: write').

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings in check-release.yml.

In the 'Remove leading v from version numbers' step (line 62–65):
  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
These values come from job outputs and are injected directly into the shell before quoting can occur.

In the 'Commit & Push changes' step (line 79–82):
  git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
  git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."
The tag argument is also unquoted (sub-rule b), allowing shell metacharacter injection.

Locations:

- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:79`
- `.github/workflows/check-release.yml:80`

### github-env-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' run: step writes values derived from needs.check-releases.outputs.* (untrusted job outputs, sourced from external API responses) directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker who can influence the firebase-tools or firebase-action release tag names could inject arbitrary environment variable definitions into subsequent steps.

Offending lines:
  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

where FIREBASE_ACTIONS_RELEASE and FIREBASE_TOOLS_RELEASE are set from ${{ needs.check-releases.outputs.* }} expressions in the same block.

Locations:

- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned all action references to full commit SHAs in build-publish.yaml (actions/checkout@v4→11d5960a, docker/metadata-action@v4→818d4b7b, docker/login-action@v2→465a0781, docker/build-push-action@v3→1104d471), docker-build-ci.yml (actions/checkout@v4→11d5960a), and check-release.yml (octokit/request-action@v2.x→02f5e7c6, madhead/semver-utils@latest→4cf918af, actions/checkout@v4→11d5960a, jacobtomlinson/gha-find-replace@v3→2ff30f64, comnoco/create-release-action@v2.0.5→7dea6dc8, docker/metadata-action@v4→818d4b7b, docker/login-action@v2→465a0781×2, docker/build-push-action@v3→1104d471). Pinned action.yaml Docker image to w9jds/firebase-action:v15.22.4@sha256:1740d8dd... preserving docker:// scheme.

2. permissions: Added top-level `permissions: contents: read` to build-publish.yaml and docker-build-ci.yml. Added top-level `permissions: contents: read` to check-release.yml; added explicit `permissions: contents: read / packages: write` to the publish job.

3. script-injection: Moved ${{ needs.check-releases.outputs.* }} expressions in the 'Remove leading v' step and 'Commit & Push changes' step into the step's env: block, referencing them as plain shell variables.

4. github-env-injection: Sanitized values written to $GITHUB_ENV using `printf '%s' ... | tr -d '\n\r'` before writing to prevent newline injection.

