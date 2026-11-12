<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.36.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.36.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action's runs.image points to a mutable tag rather than a SHA digest: `image: "docker://w9jds/firebase-action:v15.36.0"`. It should be pinned to an immutable digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-digest> # v15.36.0`.

Locations:

- `action.yaml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://w9jds/firebase-action:v15.36.0@sha256:c3b2a4b33acd85340750e0dc28eb7c1ad15b79c6beb4462d740fc5660149f68d

