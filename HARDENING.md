<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.17-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ribtoks--tdg-github-action/v0.4.17-beta** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite step references actions/setup-go by a mutable tag instead of a full 40-character commit SHA: `uses: actions/setup-go@v6`.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Sub-rule (a): the 'Build action binary' step puts an expression directly inside the run script: `env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .`. Pass it through an env var instead (for example GIT_COMMIT: ${{ github.sha }}, or the built-in $GITHUB_SHA) and expand it inside double quotes.

Locations:

- `action.yml:77`

### script-injection (severity: high)

Sub-rule (a): the 'Run tdg-github-action' step puts an expression directly inside the run script: `"${{ github.action_path }}/tdg-github-action"`. Use the built-in env var instead: `"$GITHUB_ACTION_PATH/tdg-github-action"`.

Locations:

- `action.yml:103`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 56cf7be..355dcb0 100644
--- a/action.yml
+++ b/action.yml
@@ -66,20 +66,23 @@ runs:
   using: "composite"
   steps:
     - name: Set up Go
       uses: actions/setup-go@v6
       with:
         go-version-file: ${{ github.action_path }}/go.mod
         cache: ${{ inputs.SETUP_GO_CACHE == 'true' }}
     - name: Build action binary
       shell: bash
       working-directory: ${{ github.action_path }}
+      env:
+        GIT_COMMIT: ${{ github.sha }}
       run: |
-        env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .
+        env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=$GIT_COMMIT" -o tdg-github-action .
     - id: run-tdg
       name: Run tdg-github-action
       shell: bash
       working-directory: ${{ github.workspace }}
       env:
+        ACTION_PATH: ${{ github.action_path }}
         INPUT_REPO: ${{ inputs.REPO }}
         INPUT_ISSUE_REPO: ${{ inputs.ISSUE_REPO }}
         INPUT_TOKEN: ${{ inputs.TOKEN }}
@@ -100,7 +103,7 @@ runs:
         INPUT_COMMENT_ON_ISSUES: ${{ inputs.COMMENT_ON_ISSUES }}
         INPUT_ASSIGN_FROM_BLAME: ${{ inputs.ASSIGN_FROM_BLAME }}
       run: |
-        "${{ github.action_path }}/tdg-github-action"
+        "$ACTION_PATH/tdg-github-action"
 outputs:
   scannedIssues:
     description: "Equals to 1 if completed successfully"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, prior-fix

**Notes:**

Pinned actions/setup-go to 924ae3a1cded613372ab5595356fb5720e22ba16 (v6). Moved github.sha into the GIT_COMMIT env var for the build step. The run step now calls the binary through the built-in "$GITHUB_ACTION_PATH/tdg-github-action" rather than the inline github.action_path expression. This re-applies the same security change the previous harden made: it also takes the expression out of the run script, just with the built-in variable instead of a separate ACTION_PATH env entry.

