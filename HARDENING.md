<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.20.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.20.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable tag (`docker://w9jds/firebase-action:v15.20.0`) instead of an immutable SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action reference. The image field should use a SHA256 digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://w9jds/firebase-action:v15.20.0` with the immutable SHA256 digest `docker://w9jds/firebase-action@sha256:06a24126ca68eb29c537a5530acb53f62c7b82e0d7d25a00869c5a19f595e182` in action.yaml line 14. The original tag is preserved as a comment for readability.

