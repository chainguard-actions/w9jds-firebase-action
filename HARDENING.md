<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.22.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable tag-based references instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks.

build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

check-release.yml: octokit/request-action@v2.x (×2), madhead/semver-utils@latest, actions/checkout@v4 (×2), jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

docker-build-ci.yml: actions/checkout@v4

action.yaml runs.image: docker://w9jds/firebase-action:v15.22.3 uses a mutable tag instead of a SHA digest.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:20`
- `.github/workflows/build-publish.yaml:26`
- `.github/workflows/build-publish.yaml:33`
- `.github/workflows/build-publish.yaml:40`
- `.github/workflows/build-publish.yaml:46`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:35`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:79`
- `.github/workflows/check-release.yml:86`
- `.github/workflows/check-release.yml:99`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

Two run: blocks in check-release.yml directly interpolate ${{ }} expressions into shell command strings (sub-rule a), enabling script injection.

1. 'Remove leading v from version numbers' step: shell variables are assigned directly from ${{ needs.check-releases.outputs.firebase-actions-release }} and ${{ needs.check-releases.outputs.firebase-tools-release }} inside the run: block. These outputs originate from external API calls and are not sanitized before being embedded in the shell script.

  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}

2. 'Commit & Push changes' step: ${{ needs.check-releases.outputs.firebase-tools-release }} is interpolated directly into git commit -m and git tag -a shell arguments:

  git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
  git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m ...

Locations:

- `.github/workflows/check-release.yml:68`
- `.github/workflows/check-release.yml:93`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' run: block in check-release.yml writes values derived from needs.check-releases.outputs.* (untrusted workflow-controlled data) to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The values are injected via direct ${{ }} interpolation into shell variables and then written unsanitized:

  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

An attacker who can influence the release tag name could inject newlines to set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/check-release.yml:68`

### missing-permissions (severity: medium)

Two workflow files have no top-level permissions: key and no per-job permissions: keys, meaning all jobs run with the default (potentially broad) GITHUB_TOKEN permissions.

- build-publish.yaml: no top-level or job-level permissions defined for the 'build' job.
- docker-build-ci.yml: no top-level or job-level permissions defined for the 'build' job.

Additionally, check-release.yml has no top-level permissions: key, and only the 'bump-version' job defines permissions; the 'check-releases' and 'publish' jobs have no permissions: key.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all action references to full SHA digests in build-publish.yaml (actions/checkout@v4→SHA, docker/metadata-action@v4→SHA, docker/login-action@v2→SHA ×2, docker/build-push-action@v3→SHA), check-release.yml (octokit/request-action@v2.x→SHA ×2, madhead/semver-utils@latest→SHA, actions/checkout@v4→SHA ×2, jacobtomlinson/gha-find-replace@v3→SHA, comnoco/create-release-action@v2.0.5→SHA, docker/metadata-action@v4→SHA, docker/login-action@v2→SHA ×2, docker/build-push-action@v3→SHA), docker-build-ci.yml (actions/checkout@v4→SHA). Also pinned the container image in action.yaml to docker://w9jds/firebase-action:v15.22.3@sha256:80fe64362cfc30a67eb4044df0975036928120f3d49b7077b4d91148bb7e2001.

2. **script-injection**: Moved all ${{ needs.check-releases.outputs.* }} expressions out of run: blocks into env: blocks in check-release.yml. Shell scripts now reference plain environment variables.

3. **github-env-injection**: Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing values to $GITHUB_ENV in the 'Remove leading v from version numbers' step.

4. **missing-permissions**: Added `permissions: {}` at the top level of build-publish.yaml, docker-build-ci.yml, and check-release.yml. Added job-level permissions (contents: read, packages: write as appropriate) for all jobs that lacked them.

