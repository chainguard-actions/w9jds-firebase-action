<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **w9jds--firebase-action/v15.19.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml `runs.image:` field references the Docker image `docker://w9jds/firebase-action:v15.19.0` using a mutable version tag instead of an immutable SHA digest. If the tag is overwritten (intentionally or by a supply-chain attack), the action will silently execute a different image. The image reference should use a SHA digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:15`

### script-injection (severity: high)

Rule (b) — unquoted shell variable expansion of untrusted/workflow-controlled data in entrypoint.sh:

1. Line 33: `firebase functions:config:set $CONFIG_VALUES` — `$CONFIG_VALUES` is an inherited environment variable set by the calling workflow and is expanded unquoted, allowing an attacker-controlled value to inject shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) into the command.

2. Line 35: `sh -c "firebase $*"` — the positional parameters `$*` are interpolated unquoted inside a double-quoted string passed to `sh -c`. A calling workflow can supply arguments containing shell metacharacters that will be interpreted by the inner shell, enabling command injection.

Locations:

- `entrypoint.sh:33`
- `entrypoint.sh:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. action.yaml: Pinned Docker image from mutable tag `w9jds/firebase-action:v15.19.0` to immutable digest `w9jds/firebase-action@sha256:feb4a01760bb8cb19b310281c40ad13bbdc0b54639216441e6e18c62333b9013` (tag preserved as comment). 2. entrypoint.sh line 33: Replaced unquoted `firebase functions:config:set $CONFIG_VALUES` with `printf '%s\n' "$CONFIG_VALUES" | xargs firebase functions:config:set` to safely split config values without interpreting shell metacharacters. 3. entrypoint.sh line 35: Replaced `sh -c "firebase $*"` (shell injection via unquoted positional params in double-quoted string) with `exec firebase "$@"` to pass arguments directly to firebase without an intermediate shell.

