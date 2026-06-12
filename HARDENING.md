<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.18.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.18.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image with a mutable tag reference (`docker://w9jds/firebase-action:v15.18.0`) in the `runs.image:` field. Mutable tags can be silently updated to point to a different (potentially malicious) image, enabling supply-chain attacks. The image reference should be pinned to an immutable SHA digest, e.g. `image: docker://w9jds/firebase-action@sha256:<64-hex-char-digest> # v15.18.0`.

Locations:

- `action.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from the mutable tag `docker://w9jds/firebase-action:v15.18.0` to the immutable digest `docker://w9jds/firebase-action@sha256:4257eb7dac6cfa9e773de75f0d37c5b39b8caf947a95dd91920e5b3700050962` with the original tag preserved as a comment outside the YAML quotes.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in entrypoint.sh:
1. Line 32: Added double-quotes around $CONFIG_VALUES in `firebase functions:config:set "$CONFIG_VALUES"` to prevent shell metacharacter injection from caller-supplied environment values.
2. Line 35: Replaced `sh -c "firebase $*"` with `firebase "$@"` — using `"$@"` properly quotes each positional argument individually, eliminating the injection vector from unquoted $* expansion inside a double-quoted sh -c string.

