<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.21.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple unpinned action/image references found using mutable tags instead of full SHA digests.

In action.yaml: `image: "docker://w9jds/firebase-action:v15.21.0"` uses a mutable version tag instead of a SHA digest (e.g. `docker://w9jds/firebase-action@sha256:<digest>`).

In build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

In check-release.yml: `octokit/request-action@v2.x` (×2), `madhead/semver-utils@latest`, `actions/checkout@v4` (×2), `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.

In docker-build-ci.yml: `actions/checkout@v4`.

All of these should be pinned to a full 40-character commit SHA to prevent supply-chain attacks.

Locations:

- `action.yaml:15`
- `.github/workflows/build-publish.yaml:20`
- `.github/workflows/build-publish.yaml:25`
- `.github/workflows/build-publish.yaml:33`
- `.github/workflows/build-publish.yaml:39`
- `.github/workflows/build-publish.yaml:45`
- `.github/workflows/check-release.yml:20`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:34`
- `.github/workflows/check-release.yml:38`
- `.github/workflows/check-release.yml:55`
- `.github/workflows/check-release.yml:72`
- `.github/workflows/check-release.yml:80`
- `.github/workflows/check-release.yml:91`
- `.github/workflows/check-release.yml:103`
- `.github/workflows/check-release.yml:109`
- `.github/workflows/check-release.yml:115`
- `.github/workflows/check-release.yml:122`
- `.github/workflows/docker-build-ci.yml:9`

### script-injection (severity: high)

Two `run:` blocks in check-release.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker-controlled value to be executed as shell code.

(1) The 'Remove leading v from version numbers' step interpolates `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` directly into shell variable assignments:
  `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}`
  `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}`

(2) The 'Commit & Push changes' step interpolates `${{ needs.check-releases.outputs.firebase-tools-release }}` directly into git commit message and tag arguments:
  `git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"`
  `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."`

These values come from `needs.*.outputs.*` which are workflow-controllable. They should be passed via `env:` variables and then double-quoted in the shell script.

Locations:

- `.github/workflows/check-release.yml:58`
- `.github/workflows/check-release.yml:84`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' run: block in check-release.yml writes values derived from `needs.*.outputs.*` (set via `${{ }}` template interpolation into shell variables) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

Offending lines:
  `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV`
  `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`

The shell variables `FIREBASE_ACTIONS_RELEASE` and `FIREBASE_TOOLS_RELEASE` are set directly from `${{ needs.check-releases.outputs.* }}` expressions in the same script. A value containing a newline could inject arbitrary environment variables into subsequent steps. Each write must be preceded by sanitization: `safe=$(printf '%s' "$VAR" | tr -d '\n\r')`.

Locations:

- `.github/workflows/check-release.yml:60`
- `.github/workflows/check-release.yml:61`

### missing-permissions (severity: medium)

Three workflow files are missing `permissions:` declarations, leaving them with the default (potentially broad) token permissions:

- `build-publish.yaml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.
- `docker-build-ci.yml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.
- `check-release.yml`: No top-level `permissions:` and only the `bump-version` job has a `permissions:` block (`contents: write`). The `check-releases` and `publish` jobs have no `permissions:` key, so they inherit the default broad token permissions.

All jobs should declare minimal explicit permissions (e.g. `permissions: contents: read`) to follow the principle of least privilege.

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs in action.yaml, build-publish.yaml, check-release.yml, and docker-build-ci.yml. The docker image in action.yaml was pinned with its sha256 digest while preserving the docker:// scheme and tag.

2. **script-injection**: In check-release.yml, moved `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` from inline `run:` shell code into `env:` blocks, then referenced them as plain `$FIREBASE_ACTIONS_RELEASE` / `$FIREBASE_TOOLS_RELEASE` shell variables.

3. **github-env-injection**: In the 'Remove leading v' step, sanitized the values with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_ENV` to prevent newline injection.

4. **missing-permissions**: Added top-level `permissions: contents: read` to all three workflow files. Added per-job permissions blocks: `check-releases` and `docker-build-ci build` jobs get `contents: read`; `publish` and `build-publish build` jobs get `contents: read` + `packages: write`; `bump-version` retains its existing `contents: write`.

