<!-- markdownlint-disable -->

# Hardening Report: ribtoks--tdg-github-action/v0.4.16-beta

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ribtoks--tdg-github-action/v0.4.16-beta** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references a composite step action by a mutable tag instead of a full 40-character commit SHA: `uses: actions/setup-go@v6`.

Locations:

- `action.yml:66`

### script-injection (severity: high)

Rule (a): an expression is placed straight into the 'Build action binary' run: script: `env GOFLAGS="-mod=vendor" CGO_ENABLED=0 go build -ldflags="-w -s -X main.GitCommit=${{ github.sha }}" -o tdg-github-action .`. Pass github.sha in through an env: variable (or use the built-in $GITHUB_SHA) and quote it.

Locations:

- `action.yml:73`

### script-injection (severity: high)

Rule (a): an expression is placed straight into the 'Run tdg-github-action' run: script: `"${{ github.action_path }}/tdg-github-action"`. Use the env var instead: "$GITHUB_ACTION_PATH/tdg-github-action".

Locations:

- `action.yml:99`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d270270..1423f87 100644
--- a/action.yml
+++ b/action.yml
@@ -63,19 +63,22 @@ runs:
   using: "composite"
   steps:
     - name: Set up Go
       uses: actions/setup-go@v6
       with:
         go-version-file: ${{ github.action_path }}/go.mod
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
@@ -96,7 +99,7 @@ runs:
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

I fixed all three findings in action.yml.

- **Unpinned action:** `actions/setup-go` is now pinned to the full commit `924ae3a1cded613372ab5595356fb5720e22ba16`, with a `# v6` comment so the tag stays readable.
- **Build step:** `github.sha` is now passed in through a `GIT_COMMIT` entry in the step's `env:` block, and the build command reads `$GIT_COMMIT` inside the already-quoted `-ldflags` value.
- **Run step:** the binary is now called as `"$GITHUB_ACTION_PATH/tdg-github-action"` instead of putting `github.action_path` straight into the script. The step uses bash, so this form is valid.
- **Prior fix:** the earlier harden's changes are back in place. The one difference is that I used the built-in `$GITHUB_ACTION_PATH` rather than adding a separate `ACTION_PATH` variable, which is what this finding suggested.

I haven't run the action after these changes.

