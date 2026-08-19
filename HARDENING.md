<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.19.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use unpinned (non-SHA) action references and Docker image tags, making the action vulnerable to supply-chain attacks if a tag is moved.

- action.yaml: `runs.image: docker://w9jds/firebase-action:v15.19.0` — uses a mutable tag instead of a SHA digest.
- build-publish.yaml: `actions/checkout@v4`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.
- check-release.yml: `octokit/request-action@v2.x` (×2), `madhead/semver-utils@latest`, `actions/checkout@v4`, `jacobtomlinson/gha-find-replace@v3`, `comnoco/create-release-action@v2.0.5`, `docker/metadata-action@v4`, `docker/login-action@v2` (×2), `docker/build-push-action@v3`.
- docker-build-ci.yml: `actions/checkout@v4`.

Locations:

- `action.yaml:16`
- `.github/workflows/build-publish.yaml:16`
- `.github/workflows/check-release.yml:18`
- `.github/workflows/docker-build-ci.yml:9`

### missing-permissions (severity: medium)

Several workflow files lack a top-level `permissions:` block and have jobs with no job-level `permissions:` either, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- build-publish.yaml: No top-level permissions; the single `build` job has no `permissions:` key.
- docker-build-ci.yml: No top-level permissions; the single `build` job has no `permissions:` key.
- check-release.yml: No top-level permissions; the `check-releases` and `publish` jobs have no `permissions:` key (only `bump-version` does).

Locations:

- `.github/workflows/build-publish.yaml:1`
- `.github/workflows/docker-build-ci.yml:1`
- `.github/workflows/check-release.yml:1`

### script-injection (severity: high)

check-release.yml contains two `run:` blocks that directly interpolate GitHub Actions expressions into shell commands (rule a), allowing an attacker who can influence the release tag names to inject arbitrary shell commands.

1. "Remove leading 'v' from version numbers" step (line ~62): directly interpolates `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` as bare shell assignments:
   ```
   FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
   FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
   ```

2. "Commit & Push changes" step (line ~80): directly interpolates `${{ needs.check-releases.outputs.firebase-tools-release }}` into git commit message and tag arguments:
   ```
   git commit -a -m "Bump firebase-tools to ${{ needs.check-releases.outputs.firebase-tools-release }}"
   git tag -a ${{ needs.check-releases.outputs.firebase-tools-release }} -m "..."
   ```

Locations:

- `.github/workflows/check-release.yml:62`
- `.github/workflows/check-release.yml:80`

### github-env-injection (severity: high)

In check-release.yml, the "Remove leading 'v' from version numbers" `run:` step writes values derived from `needs.check-releases.outputs.*` (which are sourced from external API responses for release tag names) to `$GITHUB_ENV` without sanitization. The values are assigned from `${{ needs.check-releases.outputs.firebase-actions-release }}` and `${{ needs.check-releases.outputs.firebase-tools-release }}` into shell variables, then written directly to $GITHUB_ENV:
```
FIREBASE_ACTIONS_RELEASE=${{ needs.check-releases.outputs.firebase-actions-release }}
FIREBASE_TOOLS_RELEASE=${{ needs.check-releases.outputs.firebase-tools-release }}
echo "FIREBASE_ACTIONS_RELEASE=${FIREBASE_ACTIONS_RELEASE#v}" >> $GITHUB_ENV
echo "FIREBASE_TOOLS_RELEASE=${FIREBASE_TOOLS_RELEASE#v}" >> $GITHUB_ENV
```
The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before each write, allowing newline injection into the environment file.

Locations:

- `.github/workflows/check-release.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across four files:

1. action.yaml: Pinned docker://w9jds/firebase-action:v15.19.0 to its SHA digest (sha256:feb4a01760bb8cb19b310281c40ad13bbdc0b54639216441e6e18c62333b9013).

2. build-publish.yaml: Pinned all 5 action references to full commit SHAs; added top-level permissions block (contents: read, packages: write).

3. docker-build-ci.yml: Pinned actions/checkout to full SHA; added top-level permissions block (contents: read).

4. check-release.yml: Pinned all 10 action references to full commit SHAs; added top-level permissions: contents: read; added permissions to publish job (contents: read, packages: write); fixed script-injection in 'Remove leading v' step and 'Commit & Push changes' step by moving ${{ }} expressions into env: blocks; fixed github-env-injection by sanitizing values with printf | tr -d '\n\r' before writing to $GITHUB_ENV.

