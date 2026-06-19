<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.22.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.22.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml references the Docker image `docker://w9jds/firebase-action:v15.22.0` using a mutable version tag instead of an immutable SHA digest. If the image at this tag is replaced, the action will silently execute different code. It should be pinned to a specific SHA digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`

Locations:

- `action.yaml:14`

### script-injection (severity: high)

Rule (b) violation: entrypoint.sh line 32 expands `$CONFIG_VALUES` unquoted in the shell command `firebase functions:config:set $CONFIG_VALUES`. `CONFIG_VALUES` is an env var inherited from the calling workflow and is therefore workflow-controlled/untrusted. An unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the value to be interpreted by the shell, enabling command injection. It should be quoted: `firebase functions:config:set "$CONFIG_VALUES"` (or handled with `--` argument separation).

Locations:

- `entrypoint.sh:32`

### unsafe-shell (severity: high)

entrypoint.sh line 35 runs `sh -c "firebase $*"`, interpolating the positional parameters (`$*`, which come from the Docker `CMD`/`args:` input) unquoted inside a double-quoted string passed to `sh -c`. This allows an attacker-controlled `args:` value containing shell metacharacters to break out of the firebase command and execute arbitrary shell commands. The safe pattern is to pass arguments as separate words: `firebase "$@"` (without `sh -c`).

Locations:

- `entrypoint.sh:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, unsafe-shell

**Notes:**

1. action.yaml line 14: Pinned Docker image from mutable tag `w9jds/firebase-action:v15.22.0` to immutable digest `w9jds/firebase-action@sha256:f9479aecc3ef32e5c699c812283f709801e9196453db8a79905772bb06ba30d8` with the version tag preserved as a comment. 2. entrypoint.sh line 32: Quoted `$CONFIG_VALUES` to prevent shell metacharacter injection: `firebase functions:config:set "$CONFIG_VALUES"`. 3. entrypoint.sh line 35: Replaced `sh -c "firebase $*"` with `firebase "$@"` to pass positional parameters as separate, properly quoted arguments without spawning a subshell, eliminating the command injection vector.

