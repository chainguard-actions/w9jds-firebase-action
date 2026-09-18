<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.30.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.30.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image referenced by a mutable tag rather than an immutable SHA digest. `image: "docker://w9jds/firebase-action:v15.30.2"` uses the tag `v15.30.2`, which can be silently overwritten to point to a different (potentially malicious) image. It should be pinned to a specific SHA256 digest, e.g. `image: "docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.30.2"`.

Locations:

- `action.yaml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from the mutable tag `docker://w9jds/firebase-action:v15.30.2` to the immutable digest `docker://w9jds/firebase-action:v15.30.2@sha256:654724e43dee252a187c541f3766329637f951fd576d69ba343a55063655d719`. The `docker://` scheme and `:v15.30.2` tag are preserved inline as required.

