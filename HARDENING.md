<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.17-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ribtoks--tdg-github-action/v0.4.17-beta** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-go@v6`, which is pinned to a mutable tag (`v6`) rather than a full 40-character commit SHA. This means the action could silently change if the tag is moved, enabling supply-chain attacks.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in two steps.

1. "Build action binary" step: `${{ github.sha }}` and `${{ github.action_path }}` are embedded directly in the shell command: `env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .`

2. "Run tdg-github-action" step: `${{ github.action_path }}` is embedded directly in the shell command: `"${{ github.action_path }}/tdg-github-action"`

Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it, bypassing shell quoting.

Locations:

- `action.yml:74`
- `action.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/setup-go@v6 to full SHA 924ae3a1cded613372ab5595356fb5720e22ba16 with '# v6' comment for readability.
2. Moved ${{ github.sha }} out of the 'Build action binary' run: block into an env: var GIT_COMMIT; the shell command now references $GIT_COMMIT.
3. Moved ${{ github.action_path }} out of the 'Run tdg-github-action' run: block into an env: var ACTION_PATH; the shell command now references "$ACTION_PATH/tdg-github-action".

