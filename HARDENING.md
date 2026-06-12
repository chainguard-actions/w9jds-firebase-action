<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.17.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.17.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The `runs.image:` field in action.yaml references a mutable Docker image tag (`docker://w9jds/firebase-action:v15.17.0`) instead of an immutable SHA digest. If the image tag is overwritten (intentionally or via a supply-chain attack), the action will silently execute different code. The reference should be pinned to a full SHA256 digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.17.0`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from `docker://w9jds/firebase-action:v15.17.0` to `docker://w9jds/firebase-action@sha256:92117f68560c7f87927a1c2ea699f46b227ec886dba238e2750e829da0aac30a` with `# v15.17.0` comment outside the YAML string quotes for readability.

