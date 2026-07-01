<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.22.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag instead of a SHA digest: `image: "docker://w9jds/firebase-action:v15.22.4"`. A mutable tag can be silently updated to point to a different (potentially malicious) image, enabling supply-chain attacks. The image reference should use a SHA digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://w9jds/firebase-action:v15.22.4` with the immutable SHA digest `docker://w9jds/firebase-action@sha256:1740d8dd69a10be1a7fe332332a9105063e2eb3c2e1a717d12a22327832dc69e` in action.yaml line 14. The original tag is preserved as a comment outside the quoted string for readability.

