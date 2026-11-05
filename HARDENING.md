<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 20 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): expressions are placed straight into the run: script in the 'Wait for preview deployment on GitHub Pages' step: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`. Pass these through env: variables and quote them.

Locations:

- `action.yml:186`

### script-injection (severity: high)

Sub-rule (a): the 'Generate comment content for deployment' step puts ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, ${{ env.action_start_time }} and ${{ inputs.qr-code }} straight into the run: script as arguments to generate-comment.sh.

Locations:

- `action.yml:196`

### script-injection (severity: high)

Sub-rule (a): expressions are placed straight into the run: script in the 'Wait for preview removal on GitHub Pages' step: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.removed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`.

Locations:

- `action.yml:257`

### script-injection (severity: high)

Sub-rule (a): the 'Generate comment content for removal' step puts ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, ${{ env.action_start_time }} and ${{ env.qr_code_provider }} straight into the run: script.

Locations:

- `action.yml:267`

### github-env-injection (severity: high)

The 'Setup preview environment' step (action.yml line 150) runs lib/main.sh. That script writes input-derived values to $GITHUB_ENV and $GITHUB_OUTPUT with nothing stripping newlines first (no printf '%s' | tr -d '\n\r'). The values are preview_file_path (from inputs.umbrella-dir and pr-number), pages_base_url (from inputs.pages-base-url or custom-url), preview_url_path, preview_url, action_repository (github.action_repository) and deployment_action (inputs.action). A value containing a newline could inject extra environment variables or outputs.

Locations:

- `lib/main.sh:51`
- `lib/main.sh:63`
- `action.yml:150`

### github-env-injection (severity: high)

The 'Generate comment content for deployment' step writes multi-line $CONTENT to $GITHUB_OUTPUT between fixed heredoc delimiters (`echo "content<<EOF"` ... `echo "EOF"`). CONTENT is built from inputs such as preview-branch, deploy-repository, qr-code and preview_url. A value containing a line reading 'EOF' ends the block early and can inject extra outputs. Use a random delimiter, e.g. one built from /dev/urandom.

Locations:

- `action.yml:207`

### github-env-injection (severity: high)

The 'Generate comment content for removal' step writes multi-line $CONTENT, which is built from input-derived values, to $GITHUB_OUTPUT between fixed heredoc delimiters (`content<<EOF` / `EOF`). A value containing a line reading 'EOF' ends the block early and can inject extra outputs.

Locations:

- `action.yml:278`

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
index 5e6751d..634fd54 100644
--- a/action.yml
+++ b/action.yml
@@ -1,293 +1,317 @@
 name: Deploy PR Preview
 author: Ross Williams
 description: >
-    Deploy a pull request preview to GitHub Pages, similar to Vercel and Netlify.
-
+  Deploy a pull request preview to GitHub Pages, similar to Vercel and Netlify.
 
 branding:
-    icon: git-pull-request
-    color: yellow
-
+  icon: git-pull-request
+  color: yellow
 inputs:
-    token:
-        description: >
-            The token to use for the deployment. Default is GITHUB_TOKEN in the current repository. If you need more permissions for things such as deploying to another repository, you can add a Personal Access Token (PAT).
-
-
-        required: false
-        default: ${{ github.token }}
-    preview-branch:
-        description: Branch on which the previews will be deployed.
-        required: false
-        default: gh-pages
-    umbrella-dir:
-        description: Path to the directory containing all previews.
-        required: false
-        default: pr-preview
-    pages-base-url:
-        description: URL of the repo's GitHub Pages site.
-        required: false
-        default: ""
-    pages-base-path:
-        description: Path that GitHub Pages is served from.
-        required: false
-        default: ""
-    source-dir:
-        description: Directory containing files to deploy.
-        required: false
-        default: .
-    pr-number:
-        description: >
-            The PR number to use for the preview path. Defaults to the PR number from the GitHub event context. Override this for testing or when not triggered by a pull_request event.
-
+  token:
+    description: >
+      The token to use for the deployment. Default is GITHUB_TOKEN in the current repository. If you need more permissions for things such as deploying to another repository, you can add a Personal Access Token (PAT).
 
-        required: false
-        default: ""
-    deploy-repository:
-        description: >
-            The GitHub repository to deploy the preview to. This should be formatted like `<org name>/<repo name>`, e.g. `rossjrw/pr-preview-action`. Defaults to the current repository.
-            
-            You will need to add a Personal Access Token (PAT) in the `token` input in order to allow the action running in one repository to make changes to another repository.
+    required: false
+    default: ${{ github.token }}
+  preview-branch:
+    description: Branch on which the previews will be deployed.
+    required: false
+    default: gh-pages
+  umbrella-dir:
+    description: Path to the directory containing all previews.
+    required: false
+    default: pr-preview
+  pages-base-url:
+    description: URL of the repo's GitHub Pages site.
+    required: false
+    default: ""
+  pages-base-path:
+    description: Path that GitHub Pages is served from.
+    required: false
+    default: ""
+  source-dir:
+    description: Directory containing files to deploy.
+    required: false
+    default: .
+  pr-number:
+    description: >
+      The PR number to use for the preview path. Defaults to the PR number from the GitHub event context. Override this for testing or when not triggered by a pull_request event.
 
+    required: false
+    default: ""
+  deploy-repository:
+    description: >
+      The GitHub repository to deploy the preview to. This should be formatted like `<org name>/<repo name>`, e.g. `rossjrw/pr-preview-action`. Defaults to the current repository.
 
-        required: false
-        default: ${{ github.repository }}
-    wait-for-pages-deployment:
-        description: >
-            Whether to wait for GitHub Pages deployment before continuing, e.g. to ensure the preview URL is actually accessible when the sticky comment is posted.
+      You will need to add a Personal Access Token (PAT) in the `token` input in order to allow the action running in one repository to make changes to another repository.
 
+    required: false
+    default: ${{ github.repository }}
+  wait-for-pages-deployment:
+    descr
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed lib/main.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/lib/main.sh b/lib/main.sh
index 3dad8ab..83bc777 100755
--- a/lib/main.sh
+++ b/lib/main.sh
@@ -35,30 +35,41 @@ action_version=$("$GITHUB_ACTION_PATH/lib/find-current-git-tag.sh" -p "$action_r
 action_start_timestamp=$(date '+%s')
 action_start_time=$(date '+%Y-%m-%d %H:%M %Z')
 
+# Sanitize values that may come from user-controlled inputs before writing to GITHUB_ENV/GITHUB_OUTPUT
+safe_pages_base_url=$(printf '%s' "$pages_base_url" | tr -d '\n\r')
+safe_preview_url_path=$(printf '%s' "$preview_url_path" | tr -d '\n\r')
+safe_preview_url=$(printf '%s' "https://$pages_base_url/$preview_url_path/" | tr -d '\n\r')
+safe_action_repository=$(printf '%s' "$action_repository" | tr -d '\n\r')
+safe_action_version=$(printf '%s' "$action_version" | tr -d '\n\r')
+safe_action_start_time=$(printf '%s' "$action_start_time" | tr -d '\n\r')
+safe_action_start_timestamp=$(printf '%s' "$action_start_timestamp" | tr -d '\n\r')
+safe_deployment_action=$(printf '%s' "$deployment_action" | tr -d '\n\r')
+safe_preview_file_path=$(printf '%s' "$preview_file_path" | tr -d '\n\r')
+
 # Export variables for later use by this action
 {
     echo "empty_dir_path=$(mktemp -d)"
-    echo "deployment_action=$deployment_action"
+    echo "deployment_action=$safe_deployment_action"
 
-    echo "preview_file_path=$preview_file_path"
-    echo "pages_base_url=$pages_base_url"
-    echo "preview_url_path=$preview_url_path"
-    echo "preview_url=https://$pages_base_url/$preview_url_path/"
+    echo "preview_file_path=$safe_preview_file_path"
+    echo "pages_base_url=$safe_pages_base_url"
+    echo "preview_url_path=$safe_preview_url_path"
+    echo "preview_url=$safe_preview_url"
 
-    echo "action_repository=$action_repository"
-    echo "action_version=$action_version"
-    echo "action_start_time=$action_start_time"
+    echo "action_repository=$safe_action_repository"
+    echo "action_version=$safe_action_version"
+    echo "action_start_time=$safe_action_start_time"
 } >> "$GITHUB_ENV"
 
 # Export variables for use by later actions in user workflow
 {
-    echo "deployment_action=$deployment_action"
+    echo "deployment_action=$safe_deployment_action"
 
-    echo "pages_base_url=$pages_base_url"
-    echo "preview_url_path=$preview_url_path"
-    echo "preview_url=https://$pages_base_url/$preview_url_path/"
+    echo "pages_base_url=$safe_pages_base_url"
+    echo "preview_url_path=$safe_preview_url_path"
+    echo "preview_url=$safe_preview_url"
 
-    echo "action_version=$action_version"
-    echo "action_start_timestamp=$action_start_timestamp"
-    echo "action_start_time=$action_start_time"
+    echo "action_version=$safe_action_version"
+    echo "action_start_timestamp=$safe_action_start_timestamp"
+    echo "action_start_time=$safe_action_start_time"
 } >> "$GITHUB_OUTPUT"

```

Locations:

- `lib/main.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

I fixed every finding in this pass and re-applied the changes from the previous harden. Nothing was run to test these changes.

**action.yml**
- The two 'Wait for ... on GitHub Pages' steps no longer put workflow values straight into the script. The repository, commit SHA, branch and token are now passed as environment variables (WAIT_DEPLOY_REPO, WAIT_COMMIT_SHA, WAIT_BRANCH, WAIT_TOKEN). The script uses them in double quotes.
- The two 'Generate comment content' steps pass all eight values to generate-comment.sh the same way, as C_* environment variables. The arguments are in the same order and quoting as upstream. The removal step still reads env.qr_code_provider, as upstream did.
- Both comment steps now end the multi-line `content` output with a delimiter made fresh on each run from /dev/urandom, instead of a fixed `EOF`. The comment text itself is written unchanged, line breaks included.

**lib/main.sh**
- Added a small `sanitize` helper (`printf '%s' | tr -d '\n\r'`). It strips line breaks from deployment_action, preview_file_path, pages_base_url, preview_url_path, preview_url, action_repository, action_version, action_start_time and action_start_timestamp before they are written to GITHUB_ENV and GITHUB_OUTPUT.
- The cleaning runs on values already stored in variables, not around a command, so a failing command can't be silently hidden by it.

