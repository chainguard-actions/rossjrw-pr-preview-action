<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.2** was hardened automatically. 20 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Wait for preview deployment on GitHub Pages' run: block puts expressions directly into the shell command: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`. The values should be passed through env: variables and quoted.

Locations:

- `action.yml:189`

### script-injection (severity: high)

Rule (a): the 'Generate comment content for deployment' run: block puts expressions directly into the generate-comment.sh call: "${{ env.action_repository }}", "${{ env.action_version }}", "${{ env.preview_url }}", "${{ inputs.preview-branch }}", "${{ github.server_url }}", "${{ inputs.deploy-repository }}", "${{ env.action_start_time }}", "${{ inputs.qr-code }}".

Locations:

- `action.yml:199`

### script-injection (severity: high)

Rule (a): the 'Wait for preview removal on GitHub Pages' run: block puts expressions directly into the shell command: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.removed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`.

Locations:

- `action.yml:260`

### script-injection (severity: high)

Rule (a): the 'Generate comment content for removal' run: block puts expressions directly into the generate-comment.sh call: "${{ env.action_repository }}", "${{ env.action_version }}", "${{ env.preview_url }}", "${{ inputs.preview-branch }}", "${{ github.server_url }}", "${{ inputs.deploy-repository }}", "${{ env.action_start_time }}", "${{ env.qr_code_provider }}".

Locations:

- `action.yml:270`

### github-env-injection (severity: high)

Case (f): the 'Generate comment content for deployment' step writes $CONTENT, which is built from inputs and env values (preview-branch, deploy-repository, qr-code, preview_url and others), to $GITHUB_OUTPUT between a fixed heredoc delimiter: `echo "content<<EOF"; echo "$CONTENT"; echo "EOF"`. If the content contains a line reading 'EOF', the block ends early and the lines after it become new outputs. The delimiter should be generated from a secure random source such as /dev/urandom on every run.

Locations:

- `action.yml:210`

### github-env-injection (severity: high)

Case (f): the 'Generate comment content for removal' step writes $CONTENT, which is built from untrusted inputs and env values, to $GITHUB_OUTPUT using the fixed heredoc delimiter 'EOF' (`echo "content<<EOF" ... echo "EOF"`).

Locations:

- `action.yml:281`

### github-env-injection (severity: high)

The 'Setup preview environment' step (action.yml line 153, `run: $GITHUB_ACTION_PATH/lib/main.sh`) writes values from inputs to $GITHUB_ENV and $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` sanitization. The inputs are umbrella-dir, pages-base-url, pages-base-path, custom-url, pr-number, deploy-repository and action, plus github.action_repository. Examples: `echo "preview_file_path=$preview_file_path"`, `echo "pages_base_url=$pages_base_url"`, `echo "deployment_action=$deployment_action"`, `echo "action_repository=$action_repository"`, with the blocks redirected `>> "$GITHUB_ENV"` (line 50) and `>> "$GITHUB_OUTPUT"` (line 62). A value containing a newline can inject other environment variables (e.g. LD_PRELOAD, BASH_ENV) or outputs.

Locations:

- `lib/main.sh:50`
- `lib/main.sh:62`
- `action.yml:153`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:200`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:202`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.qr-code }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:205`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:271`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:273`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 5e6751d..a53784a 100644
--- a/action.yml
+++ b/action.yml
@@ -134,6 +134,23 @@ outputs:
 runs:
     using: composite
     steps:
         - name: Setup preview environment
           id: setup
           env:
@@ -154,7 +171,7 @@ runs:
 
         - name: Deploy preview directory
           if: env.deployment_action == 'deploy'
           uses: JamesIves/github-pages-deploy-action@4a3abc783e1a24aeb44c16e869ad83caf6b4cc23 # v4.7.4
           with:
               token: ${{ inputs.token }}
               repository-name: ${{ inputs.deploy-repository }}
@@ -181,9 +198,14 @@ runs:
               env.deployment_action == 'deploy' &&
               (inputs.wait-for-pages-deployment == 'true' || inputs.wait-for-pages-deployment == true) &&
               env.deployment_status == 'success'
+          env:
+              DEPLOY_REPO: ${{ inputs.deploy-repository }}
+              DEPLOYED_SHA: ${{ steps.deployed-commit.outputs.deployed_commit_sha }}
+              DEPLOY_BRANCH: ${{ inputs.preview-branch }}
+              DEPLOY_TOKEN: ${{ inputs.token }}
           run: |
-              source $GITHUB_ACTION_PATH/lib/wait-for-pages-deployment.sh
-              wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"
+              source "$GITHUB_ACTION_PATH/lib/wait-for-pages-deployment.sh"
+              wait_for_pages_deployment "$DEPLOY_REPO" "$DEPLOYED_SHA" "$DEPLOY_BRANCH" "$DEPLOY_TOKEN"
           shell: bash
 
         - name: Generate comment content for deployment
@@ -192,22 +214,28 @@ runs:
               env.deployment_action == 'deploy' &&
               (inputs.comment == 'true' || inputs.comment == true) &&
               (env.deployment_status == 'success' || env.deployment_status == 'skipped')
+          env:
+              COMMENT_PREVIEW_BRANCH: ${{ inputs.preview-branch }}
+              COMMENT_SERVER_URL: ${{ github.server_url }}
+              COMMENT_DEPLOY_REPO: ${{ inputs.deploy-repository }}
+              COMMENT_QR_CODE: ${{ inputs.qr-code }}
           run: |
-              CONTENT=$($GITHUB_ACTION_PATH/lib/generate-comment.sh \
-                "${{ env.action_repository }}" \
-                "${{ env.action_version }}" \
-                "${{ env.preview_url }}" \
-                "${{ inputs.preview-branch }}" \
-                "${{ github.server_url }}" \
-                "${{ inputs.deploy-repository }}" \
-                "${{ env.action_start_time }}" \
+              CONTENT=$("$GITHUB_ACTION_PATH/lib/generate-comment.sh" \
+                "$action_repository" \
+                "$action_version" \
+                "$preview_url" \
+                "$COMMENT_PREVIEW_BRANCH" \
+                "$COMMENT_SERVER_URL" \
+                "$COMMENT_DEPLOY_REPO" \
+                "$action_start_time" \
                 "deploy" \
-                "${{ inputs.qr-code }}")
+                "$COMMENT_QR_CODE")
+              EOF_DELIM="EOF_$(od -An -N16 -tx1 /dev/urandom | tr -d ' \n')"
               {
-                echo "content<<EOF"
+                echo "content<<$EOF_DELIM"
                 echo "$CONTENT"
-                echo "EOF"
-              } >> $GITHUB_OUTPUT
+                echo "$EOF_DELIM"
+              } >> "$GITHUB_OUTPUT"
           shell: bash
 
         - name: Leave a comment after deployment
@@ -216,7 +244,7 @@ runs:
               (inputs.comment == 'true' || inputs.comment == true) &&
               (env.deployment_status == 'success' || env.deployment_status == 'skipped') &&
               steps.deploy-comment.outcome == 'success'
           uses: marocchino/sticky-pull-request-comment@67d0dec7b07ed060a405f9b2a64b8ab319fdd7db # v2.9.2
           with:
               header: pr-preview
               message: ${{ steps.deploy-comment.outputs.content }}
@@ -225,7 +253,7 @@ runs:
 
         - name: Remove preview directory
           if: env.deployment_action == 'remove'
           uses: JamesIves/github-pages-dep
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed lib/main.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/lib/main.sh b/lib/main.sh
index 3dad8ab..1409e74 100755
--- a/lib/main.sh
+++ b/lib/main.sh
@@ -35,9 +35,24 @@ action_version=$("$GITHUB_ACTION_PATH/lib/find-current-git-tag.sh" -p "$action_r
 action_start_timestamp=$(date '+%s')
 action_start_time=$(date '+%Y-%m-%d %H:%M %Z')
 
+# Strip newlines/carriage returns from values before writing to GITHUB_ENV/GITHUB_OUTPUT
+sanitize() {
+    printf '%s' "$1" | tr -d '\n\r'
+}
+deployment_action=$(sanitize "$deployment_action")
+preview_file_path=$(sanitize "$preview_file_path")
+pages_base_url=$(sanitize "$pages_base_url")
+preview_url_path=$(sanitize "$preview_url_path")
+action_repository=$(sanitize "$action_repository")
+action_version=$(sanitize "$action_version")
+action_start_timestamp=$(sanitize "$action_start_timestamp")
+action_start_time=$(sanitize "$action_start_time")
+empty_dir_path_raw=$(mktemp -d)
+empty_dir_path=$(sanitize "$empty_dir_path_raw")
+
 # Export variables for later use by this action
 {
-    echo "empty_dir_path=$(mktemp -d)"
+    echo "empty_dir_path=$empty_dir_path"
     echo "deployment_action=$deployment_action"
 
     echo "preview_file_path=$preview_file_path"

```

Locations:

- `lib/main.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

action.yml: I moved every ${{ }} expression out of the run: blocks and into env: in four steps: the two 'Wait for preview deployment/removal on GitHub Pages' steps and the two 'Generate comment content' steps. The scripts now get these values as double-quoted shell variables. The comment steps used a fixed 'EOF' delimiter when writing to $GITHUB_OUTPUT. They now use a new random delimiter from /dev/urandom on every run. The multi-line comment text is written unchanged. I also quoted the $GITHUB_ACTION_PATH script paths in the setup and wait steps. lib/main.sh: I re-applied the previous harden's change, which strips newlines and carriage returns from each value before it is written to $GITHUB_ENV or $GITHUB_OUTPUT. The mktemp result is captured on its own line first, so if mktemp fails the script still stops. lib/get-deployed-commit.sh also writes to $GITHUB_OUTPUT. I left it alone because it checks that the value is exactly 40 characters on a single line before writing it. I haven't run the action or its tests after these edits.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

In action.yml, 'Generate comment content for deployment': removed the leftover `"${{ inputs.qr-code }}")` text and the duplicate write to GITHUB_OUTPUT that used a fixed EOF delimiter. The write with a random delimiter is kept. 'Wait for preview removal on GitHub Pages': removed the duplicate wait_for_pages_deployment line that used inline expressions; the env-var version is kept. The previous pass had also cut off text in two places: the deploy-comment step's `if:` condition, and the removal-wait step's name plus its `if:` condition. I rebuilt both by copying the matching deploy/removal steps. These conditions are my reconstruction, not upstream's text, so check them against the upstream v1.8.2 action.yml. I did not run the action.

### Iteration 3

**Fixes applied:** script-injection, invalid-yaml

**Notes:**

Rebuilt the damaged removal section of action.yml so it is valid YAML again. The 'Get commit SHA of preview removal' step got back its DEPLOY_BRANCH and DEPLOY_TOKEN env vars plus its run and shell lines; lib/get-deployed-commit.sh exits with an error if either is missing. The 'Wait for preview removal on GitHub Pages' step got back its full name, if condition and env block. Deleted the duplicate wait_for_pages_deployment line that put ${{ inputs.* }} and step outputs straight into the shell; the env-based call on the line before it does the same job safely. Restored the header of the 'Generate comment content for removal' step (id remove-comment, name and if condition), which the 'Leave a comment after removal' step depends on. One behaviour change beyond the repair: that step's COMMENT_QR_CODE used to read env.qr_code_provider, which nothing in action.yml ever sets. I pointed it at inputs.qr-code, as the deploy comment step does, so removal comments now include the QR code setting.

