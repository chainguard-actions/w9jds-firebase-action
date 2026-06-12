<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.19.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.19.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image with a mutable version tag instead of an immutable SHA digest. `image: "docker://w9jds/firebase-action:v15.19.1"` references the tag `v15.19.1`, which can be silently overwritten in the registry to point to a different (potentially malicious) image. It should be pinned to a specific SHA digest, e.g. `image: "docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.19.1"`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image in action.yaml from the mutable tag `w9jds/firebase-action:v15.19.1` to the immutable digest `w9jds/firebase-action@sha256:341e7a31d5e89f76d0cfedce73ff70ea4dfeab3a13c5b0bb61787241f8f35db2`, with `# v15.19.1` preserved as a comment outside the quoted string.

