<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.27.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.27.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag/branch/version refs instead of pinned 40-character SHA commits. Unpinned actions can be silently updated to inject malicious code.

.github/workflows/build-publish.yaml:
  - actions/checkout@v4
  - docker/metadata-action@v4
  - docker/login-action@v2 (×2)
  - docker/build-push-action@v3

.github/workflows/check-release.yml:
  - octokit/request-action@v2.x (×2)
  - madhead/semver-utils@latest
  - actions/checkout@v4
  - jacobtomlinson/gha-find-replace@v3
  - comnoco/create-release-action@v2.0.5
  - docker/metadata-action@v4
  - docker/login-action@v2 (×2)
  - docker/build-push-action@v3

.github/workflows/docker-build-ci.yml:
  - actions/checkout@v4

action.yaml:
  - runs.image: docker://w9jds/firebase-action:v15.27.0 — uses a mutable version tag instead of a SHA digest (e.g., docker://w9jds/firebase-action@sha256:<digest>).

Locations:

- `.github/workflows/build-publish.yaml:19`
- `.github/workflows/build-publish.yaml:25`
- `.github/workflows/build-publish.yaml:33`
- `.github/workflows/build-publish.yaml:40`
- `.github/workflows/build-publish.yaml:47`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:35`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:68`
- `.github/workflows/check-release.yml:83`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:120`
- `.github/workflows/check-release.yml:127`
- `.github/workflows/check-release.yml:134`
- `.github/workflows/docker-build-ci.yml:9`
- `action.yaml:16`

### script-injection (severity: high)

In check-release.yml, two run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection.

(a) 'Remove leading v from version numbers' step: ${{ needs.check-releases.outputs.firebase-actions-release }} and ${{ needs.check-releases.outputs.firebase-tools-release }} are interpolated directly into shell variable assignments. An attacker who controls a release tag name could inject arbitrary shell commands.

  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}

(a) 'Commit & Push changes' step: ${{ needs.check-releases.outputs.firebase-tools-release }} is interpolated directly into git commit -m and git tag -a arguments:

  git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
  git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."

Locations:

- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:79`
- `.github/workflows/check-release.yml:80`

### github-env-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' run: step writes values derived from needs.check-releases.outputs.* (workflow-controlled, untrusted) to $GITHUB_ENV without sanitization. The values are first assigned to shell variables via direct ${{ }} interpolation, then written to $GITHUB_ENV using echo without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A malicious release tag containing newlines could inject arbitrary environment variable definitions.

  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

Locations:

- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:65`

### missing-permissions (severity: medium)

Two workflow files have no top-level permissions: key and no job-level permissions: key on every job, meaning they run with the default (potentially broad) token permissions.

- build-publish.yaml: No top-level permissions block; the single 'build' job has no permissions block.
- docker-build-ci.yml: No top-level permissions block; the single 'build' job has no permissions block.

Note: check-release.yml has a job-level permissions block on the 'bump-version' job (contents: write) but the 'check-releases' and 'publish' jobs have no permissions block, so it also fails this check.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across four files:

1. action.yaml: Pinned docker://w9jds/firebase-action:v15.27.0 to sha256:fa8b3b6d0b03b3ce4fd355b6105b063b6b85a05896964933bb326d96b5c66d90 (preserving docker:// scheme and tag).

2. build-publish.yaml: Pinned all 5 action references (actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 ×2, docker/build-push-action@v3) to full SHAs. Added top-level and job-level permissions blocks (contents: read, packages: write).

3. docker-build-ci.yml: Pinned actions/checkout@v4 to full SHA. Added top-level and job-level permissions blocks (contents: read).

4. check-release.yml: Pinned all 8 action references (octokit/request-action@v2.x ×2, madhead/semver-utils@latest, actions/checkout@v4, jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 ×2, docker/build-push-action@v3) to full SHAs. Fixed script-injection by moving ${{ needs.check-releases.outputs.* }} expressions into env: blocks. Fixed github-env-injection by sanitizing values with printf/tr before writing to $GITHUB_ENV. Added top-level permissions: contents: read and job-level permissions to check-releases and publish jobs.

