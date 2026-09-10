<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.30.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.30.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml uses `runs.image: docker://w9jds/firebase-action:v15.30.0`, which references a mutable Docker image tag (`v15.30.0`) instead of an immutable SHA digest. This means the image pulled at runtime can change without notice, enabling supply-chain attacks. It should be pinned to a specific SHA digest, e.g. `image: docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`

Locations:

- `action.yaml:16`

### script-injection (severity: high)

Sub-rule (b): entrypoint.sh passes the workflow-controlled env var `$CONFIG_VALUES` unquoted directly to the shell command `firebase functions:config:set $CONFIG_VALUES`. Because the variable is unquoted, any shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the value will be interpreted by the shell, enabling command injection. The fix is to quote the expansion: `firebase functions:config:set "$CONFIG_VALUES"`.

Locations:

- `entrypoint.sh:32`

### script-injection (severity: high)

Sub-rule (b): entrypoint.sh executes `sh -c "firebase $*"`, embedding the unquoted positional parameters (`$*`) directly inside a double-quoted string that is passed to `sh -c`. Positional parameters are supplied by the Docker CMD or the calling workflow and are untrusted; embedding them unquoted inside a shell string allows shell metacharacter injection. The fix is to pass arguments safely, e.g. `firebase "$@"` directly, or `sh -c 'firebase "$@"' -- "$@"`.

Locations:

- `entrypoint.sh:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Three fixes applied: (1) Pinned the Docker image in action.yaml from `docker://w9jds/firebase-action:v15.30.0` to `docker://w9jds/firebase-action:v15.30.0@sha256:40aa2a4696e1c3042b7330a5056e51d90edabc524b0fa60541f399c9367183a1` to prevent supply-chain attacks via mutable tags. (2) Quoted `$CONFIG_VALUES` in entrypoint.sh (`firebase functions:config:set "$CONFIG_VALUES"`) to prevent shell metacharacter injection. (3) Replaced `sh -c "firebase $*"` with `firebase "$@"` to safely pass positional arguments without shell re-interpretation, eliminating the command injection vector.

