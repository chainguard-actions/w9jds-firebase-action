<!-- markdownlint-disable -->

# Hardening Report: w9jds--firebase-action/v15.37.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **w9jds--firebase-action/v15.37.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Docker action runs.image uses a mutable tag rather than a SHA digest: `image: "docker://w9jds/firebase-action:v15.37.0"`. Pin it to the digest, e.g. docker://w9jds/firebase-action@sha256:<digest> # v15.37.0.

Locations:

- `action.yaml:15`

### script-injection (severity: high)

The action's entrypoint script runs its arguments through a shell: `sh -c "firebase $*"`. The workflow-supplied args (with: args) are joined into one string and parsed by sh. Shell metacharacters in them (;, |, &, $(...), backticks) are executed. Run the command directly with `firebase "$@"` instead.

Locations:

- `entrypoint.sh:37`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Both findings are fixed in the source, but the entrypoint.sh change does nothing at runtime yet. action.yaml still runs upstream's prebuilt image, which has the old script baked in.

- unpinned-uses: In action.yaml, runs.image is now docker://w9jds/firebase-action:v15.37.0@sha256:78ba86bede683fa0aa6bcbfdef52e705b90b42c142e1f45dd8dcb106875ecee5. The docker:// scheme and the tag are kept, and the digest came from lookup_container_digest.
- script-injection: In entrypoint.sh, `sh -c "firebase $*"` is now `firebase "$@"`. The arguments the runner passes reach the CLI one by one and are no longer re-parsed by sh.

Because the image is prebuilt, the vulnerable script still runs when the action is used. For the new script to run, the action would have to build from its own Dockerfile or use a rebuilt image; I made neither change.

Behaviour change: anyone who relied on shell features inside `with: args` (pipes, `&&`, `$(...)`) will find those characters now reach firebase as plain text instead of being run.

