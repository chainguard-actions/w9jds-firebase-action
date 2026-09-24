<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.31.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.31.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag instead of an immutable SHA digest. `image: "docker://w9jds/firebase-action:v15.31.0"` uses the tag `v15.31.0`, which can be silently overwritten by the image maintainer (or an attacker who compromises the registry account), enabling a supply-chain attack. It should be pinned to a full SHA256 digest, e.g. `image: "docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.31.0"`

Locations:

- `action.yaml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from `docker://w9jds/firebase-action:v15.31.0` to `docker://w9jds/firebase-action:v15.31.0@sha256:c8f93930f202c4fe924e8764ad861e0494456ac2a8f1830076b97022be246ea1`. The `docker://` scheme and tag are preserved; the SHA256 digest makes the reference immutable.

