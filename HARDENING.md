<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.23.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.23.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple unpinned action/image references found using mutable tags instead of full 40-character SHA commit hashes or SHA digests:
- action.yaml: `image: docker://w9jds/firebase-action:v15.23.0` (mutable tag, not a SHA digest)
- build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3
- check-release.yml: octokit/request-action@v2.x, madhead/semver-utils@latest, jacobtomlinson/gha-find-replace@v3, actions/checkout@v4 (×2), comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3
- docker-build-ci.yml: actions/checkout@v4

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:16`
- `.github/workflows/build-publish.yaml:22`
- `.github/workflows/build-publish.yaml:29`
- `.github/workflows/build-publish.yaml:36`
- `.github/workflows/build-publish.yaml:43`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:33`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:80`
- `.github/workflows/check-release.yml:97`
- `.github/workflows/check-release.yml:103`
- `.github/workflows/check-release.yml:110`
- `.github/workflows/check-release.yml:117`
- `.github/workflows/docker-build-ci.yml:10`

### missing-permissions (severity: medium)

Workflow files are missing top-level `permissions:` blocks and some jobs also lack job-level permissions, meaning the GITHUB_TOKEN is granted default (potentially write) permissions:
- build-publish.yaml: No top-level permissions and no job-level permissions on the `build` job.
- check-release.yml: No top-level permissions; the `check-releases` and `publish` jobs have no job-level permissions (only `bump-version` has `contents: write`).
- docker-build-ci.yml: No top-level permissions and no job-level permissions on the `build` job.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/check-release.yml:1`
- `.github/workflows/docker-build-ci.yml:1`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands allows script injection. An attacker who can influence the referenced values can inject arbitrary shell commands.

(a) 'Remove leading v from version numbers' step: `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}` and `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into the shell script before the shell parses it.

(a) 'Commit & Push changes' step: `git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"` and `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m ...` interpolate the expression directly into shell commands.

Locations:

- `.github/workflows/check-release.yml:60`
- `.github/workflows/check-release.yml:75`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' step in check-release.yml writes values derived from `needs.check-releases.outputs.*` (untrusted, workflow-controlled data) to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). This allows newline injection that could set arbitrary environment variables for subsequent steps.

Offending lines:
  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

The values FIREBASE_ACTIONS_RELEASE and FIREBASE_TOOLS_RELEASE are set directly from ${{ needs.check-releases.outputs.firebase-actions-release }} and ${{ needs.check-releases.outputs.firebase-tools-release }} without sanitization.

Locations:

- `.github/workflows/check-release.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across action.yaml and three workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments. Pinned docker image in action.yaml to sha256 digest. Actions pinned: actions/checkout@v4→34e114876b0b11c390a56381ad16ebd13914f8d5, docker/metadata-action@v4→818d4b7b91585d195f67373fd9cb0332e31a7175, docker/login-action@v2→465a07811f14bebb1938fbed4728c6a1ff8901fc, docker/build-push-action@v3→1104d471370f9806843c095c1db02b5a90c5f8b6, octokit/request-action@v2.x→02f5e7c637a73a3b12ed81015fa7fb5f11cc5d7d, madhead/semver-utils@latest→4cf918affe9106ea59f86c6250e5ec4570ac4389, jacobtomlinson/gha-find-replace@v3→2ff30f644d2e0078fc028beb9193f5ff0dcad39e, comnoco/create-release-action@v2.0.5→7dea6dc82ac9d97ced7a764aa82811451bba80e0.

2. missing-permissions: Added top-level 'permissions: {}' to all three workflow files. Added job-level permissions (contents: read, packages: write as needed) to all jobs that lacked them.

3. script-injection: Moved ${{ needs.check-releases.outputs.firebase-actions-release }} and ${{ needs.check-releases.outputs.firebase-tools-release }} expressions from run: shell commands into step env: blocks. References in shell now use plain $FIREBASE_ACTIONS_RELEASE and $FIREBASE_TOOLS_RELEASE environment variables.

4. github-env-injection: In the 'Remove leading v from version numbers' step, values are now sanitized using printf '%s' ... | tr -d '\n\r' before being written to $GITHUB_ENV, preventing newline injection attacks.

