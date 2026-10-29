<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.34.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.34.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The runs.image field in action.yaml uses a mutable tag instead of a fixed sha256 digest: `image: "docker://w9jds/firebase-action:v15.34.2"`. If someone with push access to that image repository re-points the tag, every run of this action will use the new image without warning. Fix: pin it to a digest, e.g. `docker://w9jds/firebase-action@sha256:<digest> # v15.34.2`.

Locations:

- `action.yaml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image in action.yaml to docker://w9jds/firebase-action:v15.34.2@sha256:ebdd206a0b1877aca2053d083a0a6802b55480625ccb7ad8678163ad1333fadf (digest from registry lookup).

