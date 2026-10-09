<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.33.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.33.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image reference with a mutable version tag (`docker://w9jds/firebase-action:v15.33.0`) instead of an immutable SHA digest. If the tag is moved or the registry is compromised, a different (potentially malicious) image could be silently pulled and executed. The image reference should be pinned to a full SHA256 digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yaml from `docker://w9jds/firebase-action:v15.33.0` to `docker://w9jds/firebase-action:v15.33.0@sha256:cf8886b558c7b2d7fadecd14e0b1dec2203eaafe31b6d7143c0fc9eaf20d78e6`. The `docker://` scheme and `:v15.33.0` tag are preserved inline, with the immutable SHA256 digest appended to prevent tag mutation attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in entrypoint.sh:
1. Line 32: Replaced unquoted `$CONFIG_VALUES` in `firebase functions:config:set $CONFIG_VALUES` with xargs-based tokenization into a bash array (`config_args`), then passed `"${config_args[@]}"` to firebase. This safely handles the space-separated list of key=value pairs without allowing shell metacharacter injection.
2. Line 35: Replaced `sh -c "firebase $*"` with `firebase "$@"`, which properly quotes each positional argument as a separate word, preventing command injection via the action's args input while correctly forwarding all arguments.

