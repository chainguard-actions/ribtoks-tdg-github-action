<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.17-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ribtoks--tdg-github-action/v0.4.17-beta** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-go@v6`, which is pinned to a mutable tag (`@v6`) rather than an immutable 40-character commit SHA. This allows the referenced action to be silently updated or replaced, enabling supply-chain attacks.

Locations:

- `action.yml:67`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ }}` expressions into shell command strings, bypassing shell quoting and enabling script injection.

1. "Build action binary" step (line ~72): `${{ github.sha }}` is interpolated directly inside the `-ldflags` shell argument:
   `env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .`

2. "run-tdg" step (line ~98): `${{ github.action_path }}` is interpolated directly as the executable path in the shell command:
   `"${{ github.action_path }}/tdg-github-action"`

Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell processes it, making it a script-injection risk.

Locations:

- `action.yml:72`
- `action.yml:98`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three issues in action.yml:
1. Pinned `actions/setup-go@v6` to full commit SHA `4a3601121dd01d1626a1e23e37211e3254c1c06c` with `# v6` comment for readability.
2. Moved `${{ github.sha }}` out of the `run:` block into an `env:` block as `GIT_COMMIT`, then referenced it as `${GIT_COMMIT}` in the `-ldflags` argument.
3. Moved `${{ github.action_path }}` out of the `run:` block into an `env:` block as `ACTION_PATH`, then referenced it as `${ACTION_PATH}` when invoking the binary.

