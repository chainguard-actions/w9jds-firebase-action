<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.21.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable tag (`docker://w9jds/firebase-action:v15.21.0`) instead of an immutable SHA digest. This means the image can be silently replaced with a different (potentially malicious) version without any change to the action reference. The image should be pinned to a specific SHA digest, e.g., `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from the mutable tag `w9jds/firebase-action:v15.21.0` to the immutable SHA digest `w9jds/firebase-action@sha256:cd45eba16db5d40397506ee249bd479768746b52c725b8c277a3eee492e657ba`. The original tag is preserved as a comment for readability.

