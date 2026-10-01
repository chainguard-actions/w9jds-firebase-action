<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.32.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.32.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yaml uses a Docker image referenced by a mutable version tag (`docker://w9jds/firebase-action:v15.32.1`) instead of an immutable SHA digest. If the image at that tag is replaced or compromised, the action will silently execute the new image. The image reference should be pinned to a full SHA256 digest, e.g. `docker://w9jds/firebase-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yaml:16`

### script-injection (severity: high)

Sub-rule (b): In `entrypoint.sh`, the environment variable `$CONFIG_VALUES` — which is inherited from (and therefore controlled by) the calling workflow — is expanded **unquoted** in the shell command `firebase functions:config:set $CONFIG_VALUES`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, etc.) out of the value, enabling command injection. The variable should be double-quoted: `firebase functions:config:set "$CONFIG_VALUES"`.

Locations:

- `entrypoint.sh:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. action.yaml: Pinned the Docker image from `docker://w9jds/firebase-action:v15.32.1` to `docker://w9jds/firebase-action:v15.32.1@sha256:b46fd2315fef44b2ca6841b9d0b8f8062bbafc75662f0c1adf599d234120698c`, preserving the `docker://` scheme and the tag for readability.
2. entrypoint.sh: Fixed the unquoted `$CONFIG_VALUES` expansion on line 33. Since CONFIG_VALUES is an args-style input (space-separated key=value pairs for firebase functions:config:set), used the xargs-based tokenization pattern to safely split it into a bash array, then expanded the array with `"${config_args[@]}"`. This prevents shell metacharacter injection while preserving correct argument boundaries. The script has a `#!/bin/bash` shebang so bash arrays and process substitution are available.

