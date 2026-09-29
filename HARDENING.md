<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.32.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.32.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag rather than an immutable SHA digest. `image: "docker://w9jds/firebase-action:v15.32.0"` uses the tag `v15.32.0`, which can be silently overwritten to point to a different (potentially malicious) image. It should be pinned to a specific SHA digest, e.g. `image: "docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.32.0"`.

Locations:

- `action.yaml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from the mutable tag `w9jds/firebase-action:v15.32.0` to the immutable digest `w9jds/firebase-action:v15.32.0@sha256:098916a7f2efc541cb18ef597720cb0715c9620e15a4956af1b4b607ce93acb4`. The `docker://` scheme and the tag are preserved inline as required.

