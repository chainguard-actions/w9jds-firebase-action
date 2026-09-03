<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.29.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.29.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple unpinned action/image references found.

In action.yaml: `runs.image: docker://w9jds/firebase-action:v15.29.0` uses a mutable tag instead of a SHA digest (e.g. `docker://w9jds/firebase-action@sha256:<digest>`).

In .github/workflows/build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all use version tags, not 40-char commit SHAs.

In .github/workflows/check-release.yml: `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all use version tags or branch names.

In .github/workflows/docker-build-ci.yml: `actions/checkout@v4` uses a version tag.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:17`
- `.github/workflows/build-publish.yaml:22`
- `.github/workflows/build-publish.yaml:28`
- `.github/workflows/build-publish.yaml:34`
- `.github/workflows/build-publish.yaml:40`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:32`
- `.github/workflows/check-release.yml:55`
- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:79`
- `.github/workflows/check-release.yml:96`
- `.github/workflows/check-release.yml:103`
- `.github/workflows/check-release.yml:109`
- `.github/workflows/check-release.yml:115`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands in check-release.yml.

(a) Sub-rule a — direct interpolation:
- 'Remove leading v' step: `FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}` and `FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variable assignments. An attacker-controlled release tag containing shell metacharacters (`;`, `$(...)`, etc.) would be executed by the shell.
- 'Commit & Push changes' step: `git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"` and `git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m ...` interpolate the output directly into shell commands. The `git tag` line is also unquoted.

(b) Sub-rule b — unquoted shell variable expansion:
- In the 'Remove leading v' step, `${FIREBASE_ACTIONS_RELEASE#v}` and `${FIREBASE_TOOLS_RELEASE#v}` are used inside echo without double-quoting the surrounding expansion.

Locations:

- `.github/workflows/check-release.yml:52`
- `.github/workflows/check-release.yml:53`
- `.github/workflows/check-release.yml:72`
- `.github/workflows/check-release.yml:73`

### github-env-injection (severity: high)

The 'Remove leading v from version numbers' step in check-release.yml writes values derived from `needs.check-releases.outputs.*` (workflow-controlled, untrusted) to $GITHUB_ENV without sanitization. The expressions `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` are interpolated directly into shell variables, which are then written to $GITHUB_ENV via `echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV` and `echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing a malicious release tag containing newlines to inject arbitrary environment variables.

Locations:

- `.github/workflows/check-release.yml:54`
- `.github/workflows/check-release.yml:55`

### missing-permissions (severity: medium)

Workflow files are missing top-level or per-job `permissions:` blocks.

- build-publish.yaml: No top-level `permissions:` key and the single `build` job has no `permissions:` key. This means the job runs with the default (broad) GITHUB_TOKEN permissions.
- docker-build-ci.yml: No top-level `permissions:` key and the single `build` job has no `permissions:` key.
- check-release.yml: No top-level `permissions:` key, and the `check-releases` job and `publish` job have no `permissions:` key (only `bump-version` has `permissions: contents: write`).

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs (actions/checkout, docker/metadata-action, docker/login-action, docker/build-push-action, octokit/request-action, madhead/semver-utils, jacobtomlinson/gha-find-replace, comnoco/create-release-action). Pinned the docker://w9jds/firebase-action:v15.29.0 image to its sha256 digest while preserving the docker:// scheme and tag.

2. script-injection: In check-release.yml, moved all ${{ needs.check-releases.outputs.* }} expressions from run: blocks into env: blocks, then referenced them as shell variables (${VAR_NAME}) in the run: scripts. The git tag command now properly double-quotes the variable.

3. github-env-injection: In check-release.yml's 'Remove leading v' step, added printf '%s' ... | tr -d '\n\r' sanitization before writing values to $GITHUB_ENV to prevent newline injection.

4. missing-permissions: Added top-level permissions: blocks to build-publish.yaml (contents: read, packages: write), docker-build-ci.yml (contents: read), and check-release.yml (contents: read). Added per-job permissions to the publish job in check-release.yml (contents: read, packages: write).

