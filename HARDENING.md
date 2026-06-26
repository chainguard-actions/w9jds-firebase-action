<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.22.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable tag (`docker://w9jds/firebase-action:v15.22.3`) instead of an immutable SHA digest. This means the image could be silently replaced with a different (potentially malicious) version without any change to the action reference. It should be pinned to a SHA digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.22.3`.

Locations:

- `action.yaml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from the mutable tag `docker://w9jds/firebase-action:v15.22.3` to the immutable digest `docker://w9jds/firebase-action@sha256:80fe64362cfc30a67eb4044df0975036928120f3d49b7077b4d91148bb7e2001` with the original tag preserved as a comment.

