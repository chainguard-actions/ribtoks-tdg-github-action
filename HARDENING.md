<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.16-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ribtoks--tdg-github-action/v0.4.16-beta** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-go@v6` which is pinned to a mutable version tag (`@v6`) rather than an immutable 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved to a different commit.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, which is a script injection risk as the values are substituted by the YAML template engine before the shell ever sees them. (1) Line 68: `go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}"` — ${{ github.sha }} is interpolated directly into the shell command; use $GITHUB_SHA instead. (2) Line 88: `"${{ github.action_path }}/tdg-github-action"` — ${{ github.action_path }} is interpolated directly into the shell command; use $GITHUB_ACTION_PATH instead.

Locations:

- `action.yml:68`
- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three issues in action.yml: (1) Pinned actions/setup-go@v6 to its full commit SHA 4a3601121dd01d1626a1e23e37211e3254c1c06c with a # v6 comment for readability. (2) Replaced ${{ github.sha }} in the go build ldflags with $GITHUB_SHA, which is a built-in GitHub Actions environment variable that is safe to use directly in shell without template injection risk. (3) Replaced ${{ github.action_path }} in the binary execution command with $GITHUB_ACTION_PATH, which is also a built-in environment variable safe for direct shell use.

