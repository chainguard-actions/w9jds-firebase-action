<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.22.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable tag/branch refs instead of pinned full SHA commits, making them vulnerable to supply-chain attacks.

**action.yaml**: `runs.image: docker://w9jds/firebase-action:v15.22.2` — uses a mutable Docker tag instead of a SHA digest.

**build-publish.yaml**: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all tag-based.

**check-release.yml**: `octokit/request-action@v2.x`, `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3` — all tag/branch-based.

**docker-build-ci.yml**: `actions/checkout@v4` — tag-based.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:19`
- `.github/workflows/build-publish.yaml:24`
- `.github/workflows/build-publish.yaml:31`
- `.github/workflows/build-publish.yaml:37`
- `.github/workflows/build-publish.yaml:43`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/check-release.yml:26`
- `.github/workflows/check-release.yml:32`
- `.github/workflows/check-release.yml:57`
- `.github/workflows/check-release.yml:72`
- `.github/workflows/check-release.yml:88`
- `.github/workflows/check-release.yml:101`
- `.github/workflows/check-release.yml:113`
- `.github/workflows/check-release.yml:120`
- `.github/workflows/check-release.yml:126`
- `.github/workflows/check-release.yml:132`
- `.github/workflows/docker-build-ci.yml:10`

### script-injection (severity: high)

In `.github/workflows/check-release.yml`, two `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, enabling script injection.

**Step "Remove leading 'v' from version numbers"** (sub-rule a): `needs.*.outputs.*` values are interpolated directly into shell variable assignments:
```
FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
```
An attacker who controls a release tag name could inject arbitrary shell commands.

**Step "Commit & Push changes"** (sub-rule a): `needs.*.outputs.*` values are interpolated directly into `git commit -m` and `git tag -a` arguments:
```
git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."
```
The `git tag -a` line is also unquoted (sub-rule b).

Locations:

- `.github/workflows/check-release.yml:63`
- `.github/workflows/check-release.yml:64`
- `.github/workflows/check-release.yml:82`
- `.github/workflows/check-release.yml:83`

### github-env-injection (severity: high)

In `.github/workflows/check-release.yml`, the "Remove leading 'v' from version numbers" `run:` block writes values derived from `needs.check-releases.outputs.*` (untrusted `needs.*.outputs.*` context) to `$GITHUB_ENV` without sanitization (`printf '%s' ... | tr -d '\n\r'`). The values are first assigned to shell variables via direct `${{ ... }}` interpolation, then written to `$GITHUB_ENV`:
```bash
FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV
```
A newline embedded in a release tag name could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/check-release.yml:65`
- `.github/workflows/check-release.yml:66`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on every job, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- `build-publish.yaml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.
- `docker-build-ci.yml`: No top-level `permissions:` and the single `build` job has no `permissions:` key.

(Note: `check-release.yml` has a job-level `permissions: contents: write` on `bump-version` but the `check-releases` and `publish` jobs have no `permissions:` key, so it also fails.)

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/check-release.yml:1`
- `.github/workflows/docker-build-ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across four files:

1. **action.yaml**: Pinned Docker image `w9jds/firebase-action:v15.22.2` to its SHA digest (preserving tag inline).

2. **build-publish.yaml**: Pinned all 5 action refs to full commit SHAs with tag comments. Added top-level `permissions: contents: read, packages: write`.

3. **docker-build-ci.yml**: Pinned `actions/checkout@v4` to full commit SHA. Added top-level `permissions: contents: read`.

4. **check-release.yml**: 
   - Pinned all 11 action refs to full commit SHAs with tag comments.
   - Added top-level `permissions: contents: read` and job-level permissions on `check-releases` and `publish` jobs.
   - Fixed script injection in "Remove leading 'v'" step: moved `${{ needs.check-releases.outputs.* }}` expressions into `env:` block.
   - Fixed github-env-injection: sanitized values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.
   - Fixed script injection in "Commit & Push changes" step: moved `${{ needs.check-releases.outputs.firebase-tools-release }}` into `env:` block and referenced as `$FIREBASE_TOOLS_RELEASE`; also properly quoted the `git tag -a` argument.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection vulnerabilities in hardened/action/entrypoint.sh: (1) Quoted `$CONFIG_VALUES` → `"$CONFIG_VALUES"` on line 33 to prevent word-splitting and glob expansion of attacker-controlled input. (2) Replaced `sh -c "firebase $*"` with `firebase "$@"` on line 36 to eliminate the extra shell interpretation layer and safely pass positional arguments without risk of shell metacharacter injection via `$*` inside a double-quoted `sh -c` string.

