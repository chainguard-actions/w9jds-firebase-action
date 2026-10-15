<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.34.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.34.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag (`docker://w9jds/firebase-action:v15.34.0`) instead of an immutable SHA digest. If the tag is moved or the registry is compromised, the action will silently execute a different (potentially malicious) image. The image reference should be pinned to a full SHA256 digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from `docker://w9jds/firebase-action:v15.34.0` to `docker://w9jds/firebase-action:v15.34.0@sha256:c57e1ad457845b2a4d3ade789f3143c9da73b939ae75ec812661e8354e2b23bd`. The `docker://` scheme and `:v15.34.0` tag are preserved inline alongside the digest for readability and correctness.

