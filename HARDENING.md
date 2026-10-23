<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.34.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.34.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

This Docker action's runs.image is pinned to a mutable tag, not a SHA digest: `image: "docker://w9jds/firebase-action:v15.34.1"`. Change it to `docker://w9jds/firebase-action@sha256:<digest> # v15.34.1`.

Locations:

- `action.yaml:15`

### script-injection (severity: high)

The entrypoint script passes its arguments to a shell: `sh -c "firebase $*"`. The calling workflow supplies these arguments through `with: args:`, and they often contain expressions such as inputs or github context. Because sh -c parses them again, shell metacharacters (;, |, $(...), backticks) in the args are executed as commands. Call `firebase "$@"` directly instead of going through sh -c.

Locations:

- `entrypoint.sh:37`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

action.yaml: runs.image is now pinned to a digest, with the docker:// scheme and the tag kept in place: docker://w9jds/firebase-action:v15.34.1@sha256:61cf6e97f8a87a401ecdbc6bef07b2eec981e58f9b5490ed2fe793885d409ffc. entrypoint.sh: changed `sh -c "firebase $*"` to `firebase "$@"`. The runner's split of args is now passed straight through and no shell re-reads it. Caveat: the action still runs upstream's prebuilt image, and that image contains upstream's own entrypoint.sh. So the entrypoint fix only takes effect once the image is rebuilt from this Dockerfile, or if runs.image is switched to 'Dockerfile'. I did not make that switch.

