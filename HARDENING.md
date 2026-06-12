<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.16.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.16.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable tag (`docker://w9jds/firebase-action:v15.16.0`) under `runs.image:` instead of an immutable SHA digest. A mutable tag can be silently updated to point to a different (potentially malicious) image, enabling supply-chain attacks. It should be replaced with a digest-pinned reference such as `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://w9jds/firebase-action:v15.16.0` with the immutable digest-pinned reference `docker://w9jds/firebase-action@sha256:bbd3e753c68a922db946027dce342491753f1b33b71cb5321dfd4869acbcfbe3` in action.yaml line 14. The original tag `v15.16.0` is preserved as a comment for readability.

