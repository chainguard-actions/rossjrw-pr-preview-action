<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.2** was hardened automatically. 18 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Wait for preview deployment on GitHub Pages' puts expressions directly into the run: script. Offending line: wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}". These values should be passed through env: variables and expanded with double quotes.

Locations:

- `action.yml:186`

### script-injection (severity: high)

Rule (a): the step 'Generate comment content for deployment' puts expressions directly into the run: script as arguments to generate-comment.sh: "${{ env.action_repository }}", "${{ env.action_version }}", "${{ env.preview_url }}", "${{ inputs.preview-branch }}", "${{ github.server_url }}", "${{ inputs.deploy-repository }}", "${{ env.action_start_time }}", "${{ inputs.qr-code }}".

Locations:

- `action.yml:196`

### script-injection (severity: high)

Rule (a): the step 'Wait for preview removal on GitHub Pages' puts expressions directly into the run: script. Offending line: wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.removed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}".

Locations:

- `action.yml:257`

### script-injection (severity: high)

Rule (a): the step 'Generate comment content for removal' puts expressions directly into the run: script as arguments to generate-comment.sh: "${{ env.action_repository }}", "${{ env.action_version }}", "${{ env.preview_url }}", "${{ inputs.preview-branch }}", "${{ github.server_url }}", "${{ inputs.deploy-repository }}", "${{ env.action_start_time }}", "${{ env.qr_code_provider }}".

Locations:

- `action.yml:267`

### github-env-injection (severity: high)

The step 'Generate comment content for deployment' writes CONTENT to $GITHUB_OUTPUT using a fixed 'EOF' heredoc delimiter, with no printf '%s' | tr -d '\n\r' step first. CONTENT is built from inputs.* and env.* values (preview-branch, deploy-repository, qr-code, preview_url). A crafted input that contains a line reading 'EOF' can end the heredoc early and inject extra outputs.

Locations:

- `action.yml:210`

### github-env-injection (severity: high)

The step 'Generate comment content for removal' writes CONTENT to $GITHUB_OUTPUT using a fixed 'EOF' heredoc delimiter, with no sanitization. CONTENT is built from inputs.* and env.* values (preview-branch, deploy-repository, preview_url, qr_code_provider).

Locations:

- `action.yml:281`

### github-env-injection (severity: high)

lib/main.sh, run by the 'Setup preview environment' step (action.yml line 150), writes values built from inputs to $GITHUB_ENV and $GITHUB_OUTPUT without printf '%s' | tr -d '\n\r'. These inputs are umbrella-dir, pr-number, pages-base-url, pages-base-path, custom-url, deploy-repository and action_repository. The writes include preview_file_path, pages_base_url, preview_url_path, preview_url and action_repository. A newline in one of these inputs can inject arbitrary environment variables or outputs, for example a malicious deployment_action, preview_file_path or BASH_ENV.

Locations:

- `lib/main.sh:51`
- `lib/main.sh:63`
- `action.yml:150`

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

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

action.yml: In the two 'Wait for ... on GitHub Pages' steps, the deploy-repository, commit SHA, preview-branch and token values are now passed through env: (DEPLOY_REPO, DEPLOYED_SHA, DEPLOY_BRANCH, DEPLOY_TOKEN) and expanded in double quotes. In the two 'Generate comment content' steps, the input and github values now come through COMMENT_* env vars. The env.* values that main.sh writes to GITHUB_ENV are read as shell variables ($action_repository, $action_version, $preview_url, $action_start_time). The content output now uses a random delimiter from /dev/urandom instead of a fixed EOF, and the multi-line comment is written unchanged. lib/main.sh: added a sanitize() helper (printf '%s' | tr -d '\n\r') and run every value through it before writing to GITHUB_ENV or GITHUB_OUTPUT. The mktemp result is captured first and then sanitized, so a mktemp failure still stops the step under errexit.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

In 'Generate comment content for removal', moved all ${{ }} expressions into the step env: block and referenced them as double-quoted shell variables; replaced the fixed 'EOF' heredoc delimiter with a random per-run delimiter from /dev/urandom (multi-line content preserved), and quoted $GITHUB_OUTPUT and the script path.

