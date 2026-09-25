<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.16-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ribtoks--tdg-github-action/v0.4.16-beta** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-go@v6`, which is pinned to a mutable tag (`v6`) rather than a full 40-character commit SHA. This means the referenced action can be silently changed by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Sub-rule (a): The 'Build action binary' run block directly interpolates `${{ github.sha }}` inside the shell command string: `go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .`. Any `${{ }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yml:68`

### script-injection (severity: high)

Sub-rule (a): The 'Run tdg-github-action' run block directly interpolates `${{ github.action_path }}` inside the shell command string: `"${{ github.action_path }}/tdg-github-action"`. The `github.action_path` context value is substituted by the YAML template engine before the shell executes, meaning a specially crafted path could inject shell metacharacters.

Locations:

- `action.yml:89`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three findings in hardened/action/action.yml: (1) Pinned actions/setup-go@v6 to full SHA 924ae3a1cded613372ab5595356fb5720e22ba16 with tag comment. (2) Moved ${{ github.sha }} into env var GIT_COMMIT in the 'Build action binary' step, referencing it as $GIT_COMMIT in the shell command. (3) Moved ${{ github.action_path }} into env var ACTION_PATH in the 'Run tdg-github-action' step, referencing it as "$ACTION_PATH/tdg-github-action" in the shell command. The existing INPUT_* env vars in the run-tdg step were preserved and merged into a single env block.

