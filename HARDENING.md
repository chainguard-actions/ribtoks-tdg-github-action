<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.13-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ribtoks--tdg-github-action/v0.4.13-beta** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced action is compromised or its tag is moved.

- `.github/workflows/go.yml`: `uses: actions/setup-go@v5` (line 19), `uses: actions/checkout@v4` (line 23)
- `.github/workflows/integration.yml`: `uses: actions/checkout@master` (line 18), `uses: ribtoks/tdg-github-action@master` (line 22)

All should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/go.yml:19`
- `.github/workflows/go.yml:23`
- `.github/workflows/integration.yml:18`
- `.github/workflows/integration.yml:22`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and neither of their jobs defines a job-level `permissions:` block. Without explicit permissions, the default GITHUB_TOKEN permissions (which can be broad, especially on older repositories) are used, violating the principle of least privilege.

Locations:

- `.github/workflows/go.yml:1`
- `.github/workflows/integration.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string in `.github/workflows/integration.yml` (line 32). The offending line is:

  `test "${{ steps.selftest.outputs.scannedIssues }}" == "1"`

`steps.selftest.outputs.scannedIssues` is a workflow-controllable value that flows through YAML template substitution before the shell sees it. An attacker who can influence the action's output (e.g. via a crafted repository with a malicious TODO comment) could inject arbitrary shell commands. The value should be passed via an `env:` variable and the shell variable should be double-quoted instead.

Locations:

- `.github/workflows/integration.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/go.yml and .github/workflows/integration.yml:

1. **unpinned-uses**: Pinned all four action references to full SHAs:
   - actions/setup-go@v5 → @40f1582b2485089dde7abd97c1529aa768e1baff # v5
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/checkout@master → @61b9e3751b92087fd0b06925ba6dd6314e06f089 # master
   - ribtoks/tdg-github-action@master → @2e71a53c3aae01ab7196b87f3575739462cb507d # master

2. **missing-permissions**: Added `permissions: {}` at the top level of both workflow files, and `permissions: { contents: read }` at the job level (minimum needed for checkout and build).

3. **script-injection**: Moved `${{ steps.selftest.outputs.scannedIssues }}` out of the `run:` shell string in integration.yml into an `env:` block as `SCANNED_ISSUES`, and updated the shell script to reference `"$SCANNED_ISSUES"` instead.

