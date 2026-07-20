<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.23.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.23.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag-based `uses:` references instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced action tag is moved or compromised.

build-publish.yaml: actions/checkout@v4, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

check-release.yml: octokit/request-action@v2.x (×2), madhead/semver-utils@latest, actions/checkout@v4 (×2), jacobtomlinson/gha-find-replace@v3, comnoco/create-release-action@v2.0.5, docker/metadata-action@v4, docker/login-action@v2 (×2), docker/build-push-action@v3

docker-build-ci.yml: actions/checkout@v4

action.yaml: The docker image reference `docker://w9jds/firebase-action:v15.23.0` uses a mutable tag instead of a SHA digest.

Locations:

- `.github/workflows/build-publish.yaml:17`
- `.github/workflows/build-publish.yaml:23`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:39`
- `.github/workflows/build-publish.yaml:47`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:33`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:76`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:120`
- `.github/workflows/check-release.yml:128`
- `.github/workflows/check-release.yml:135`
- `.github/workflows/check-release.yml:141`
- `.github/workflows/docker-build-ci.yml:9`
- `action.yaml:14`

### script-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' step directly interpolates `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` (values derived from external GitHub API release tag names) into a `run:` shell script without routing through env vars. This is a rule-(a) violation: any `${{ ... }}` expression inside a `run:` block is a script-injection risk.

Offending lines:
  FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
  FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}

Additionally, the 'Commit & Push changes' step directly interpolates `${{ needs.check-releases.outputs.firebase-tools-release }}` into git commands:
  git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
  git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."

The `git tag -a ${{ ... }}` form is especially dangerous as the value is unquoted, allowing shell metacharacter injection.

Locations:

- `.github/workflows/check-release.yml:70`
- `.github/workflows/check-release.yml:71`
- `.github/workflows/check-release.yml:88`
- `.github/workflows/check-release.yml:89`

### github-env-injection (severity: high)

In check-release.yml, the 'Remove leading v from version numbers' step writes values derived from `needs.check-releases.outputs.*` (external GitHub API release tag names) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence a release tag name on either the firebase-action or firebase-tools repositories could inject arbitrary environment variables into subsequent steps.

Offending lines:
  echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
  echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV

The variables `FIREBASE_ACTIONS_RELEASE` and `FIREBASE_TOOLS_RELEASE` are set directly from `${{ needs.check-releases.outputs.* }}` expressions in the same run block, making them untrusted inputs written unsanitized to GITHUB_ENV.

Locations:

- `.github/workflows/check-release.yml:72`
- `.github/workflows/check-release.yml:73`

### missing-permissions (severity: medium)

Multiple workflow files lack a top-level `permissions:` block, and not every job within them defines its own `permissions:` block. This means jobs run with the default (potentially broad) GITHUB_TOKEN permissions.

- build-publish.yaml: No top-level `permissions:` and no job-level `permissions:` on any job.
- docker-build-ci.yml: No top-level `permissions:` and no job-level `permissions:` on any job.
- check-release.yml: No top-level `permissions:`; only the `bump-version` job has a `permissions:` block. The `check-releases` and `publish` jobs have no `permissions:` block.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across four files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs with tag comments:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - docker/metadata-action@v4 → @818d4b7b91585d195f67373fd9cb0332e31a7175
   - docker/login-action@v2 → @465a07811f14bebb1938fbed4728c6a1ff8901fc
   - docker/build-push-action@v3 → @1104d471370f9806843c095c1db02b5a90c5f8b6
   - octokit/request-action@v2.x → @02f5e7c637a73a3b12ed81015fa7fb5f11cc5d7d
   - madhead/semver-utils@latest → @4cf918affe9106ea59f86c6250e5ec4570ac4389
   - jacobtomlinson/gha-find-replace@v3 → @2ff30f644d2e0078fc028beb9193f5ff0dcad39e
   - comnoco/create-release-action@v2.0.5 → @7dea6dc82ac9d97ced7a764aa82811451bba80e0
   - action.yaml docker image pinned with sha256 digest (preserving docker:// scheme and tag)

2. **script-injection**: Moved `${{ needs.check-releases.outputs.* }}` expressions in check-release.yml 'Remove leading v' and 'Commit & Push changes' steps into env: blocks; referenced as plain shell variables (quoted) in run: scripts.

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV in the 'Remove leading v' step.

4. **missing-permissions**: Added top-level `permissions: {}` to all three workflow files. Added job-level permissions to check-releases (contents: read), publish (contents: read, packages: write), and build jobs in build-publish.yaml and docker-build-ci.yml.

