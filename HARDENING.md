<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.28.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.28.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple action references and the Docker image in action.yaml are pinned to mutable tags/branches rather than immutable full-length commit SHAs or digest references, making the action vulnerable to supply-chain attacks.

action.yaml: `image: "docker://w9jds/firebase-action:v15.28.2"` — mutable tag, not a SHA digest.

build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

check-release.yml: `octokit/request-action@v2.x`, `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4` (×2), `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

docker-build-ci.yml: `actions/checkout@v4`.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:18`
- `.github/workflows/build-publish.yaml:23`
- `.github/workflows/build-publish.yaml:30`
- `.github/workflows/build-publish.yaml:36`
- `.github/workflows/build-publish.yaml:42`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:31`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:68`
- `.github/workflows/check-release.yml:90`
- `.github/workflows/check-release.yml:100`
- `.github/workflows/check-release.yml:107`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:119`
- `.github/workflows/docker-build-ci.yml:11`

### script-injection (severity: high)

Two `run:` blocks in check-release.yml directly interpolate `${{ needs.check-releases.outputs.* }}` expressions (workflow-controlled `needs.*.outputs.*` values) into shell command strings, violating sub-rule (a).

**Step: "Remove leading 'v' from version numbers"** — the expressions `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments. An attacker who can influence the release tag name could inject arbitrary shell commands.

**Step: "Commit & Push changes"** — the expression `${{ needs.check-releases.outputs.firebase-tools-release }}` is interpolated directly into `git commit -a -m "..."` and `git tag -a ${{ ... }}` shell commands without quoting or sanitization.

Locations:

- `.github/workflows/check-release.yml:61`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:83`
- `.github/workflows/check-release.yml:84`

### github-env-injection (severity: high)

The "Remove leading 'v' from version numbers" step in check-release.yml writes `needs.*.outputs.*` values to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The values from `needs.check-releases.outputs.firebase-actions-release` and `needs.check-releases.outputs.firebase-tools-release` are assigned to shell variables via direct `${{ ... }}` interpolation and then written to `$GITHUB_ENV` with `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV` and `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`. A newline embedded in the release tag value could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`

### missing-permissions (severity: medium)

Three workflow files lack a top-level `permissions:` block, and not every job within them defines its own `permissions:` block, meaning jobs run with the default (potentially broad) token permissions.

- `build-publish.yaml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.
- `check-release.yml`: No top-level `permissions:`; the `check-releases` job and the `publish` job have no `permissions:` key (only `bump-version` has `permissions: contents: write`).
- `docker-build-ci.yml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/check-release.yml:1`
- `.github/workflows/docker-build-ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs and the Docker image in action.yaml to its sha256 digest (preserving the docker:// scheme and tag inline).

2. **script-injection**: In check-release.yml, moved `needs.check-releases.outputs.*` expressions out of `run:` shell strings into `env:` blocks, then referenced them as plain environment variables ($FIREBASE_ACTIONS_RELEASE_RAW, $FIREBASE_TOOLS_RELEASE_RAW, $FIREBASE_TOOLS_RELEASE) in the shell scripts.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values to $GITHUB_ENV in the 'Remove leading v' step.

4. **missing-permissions**: Added `permissions: {}` top-level blocks to all three workflow files (build-publish.yaml, check-release.yml, docker-build-ci.yml), and added minimal job-level permissions (contents: read, packages: write as appropriate) to each job.

