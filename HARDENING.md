<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.35.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.35.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action's runs.image points to a mutable tag instead of a SHA256 digest: `image: "docker://w9jds/firebase-action:v15.35.0"`. Pin it to a digest, for example docker://w9jds/firebase-action@sha256:<digest> # v15.35.0.

Locations:

- `action.yaml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://w9jds/firebase-action:v15.35.0@sha256:289d4f224e4d31ce71b57eed0d7cd010031bc9b948d841d5a6838b6697a1f153

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

entrypoint.sh: replaced `sh -c "firebase $*"` with `firebase "$@"`, so the container arguments go straight to the CLI and no shell re-parses them. The runner already splits `with: args:` into argv, quotes included, so normal commands like `deploy --only hosting` behave the same. Caveat: action.yaml runs the prebuilt image docker://w9jds/firebase-action:v15.35.0@sha256:289d…, not the local Dockerfile. This edit only takes effect once that image is rebuilt from the fixed entrypoint.sh, or once runs.image points at the Dockerfile.

