<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.30.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.30.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable tag instead of an immutable SHA digest. `image: docker://w9jds/firebase-action:v15.30.1` can be silently replaced by a malicious image pushed to the same tag, enabling a supply-chain attack. It should be pinned to a full SHA256 digest, e.g. `image: docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.30.1`.

Locations:

- `action.yaml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from `docker://w9jds/firebase-action:v15.30.1` to `docker://w9jds/firebase-action:v15.30.1@sha256:f4dd5ef8027ff2cfd8016e3db40497e43177121407e88d000f59c32b19928196`. The docker:// scheme and tag are preserved inline, with the immutable SHA256 digest appended to prevent supply-chain attacks via mutable tag replacement.

