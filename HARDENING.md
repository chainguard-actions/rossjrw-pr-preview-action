<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.3** was hardened automatically. 15 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. The 'Wait for preview deployment on GitHub Pages' step interpolates ${{ inputs.deploy-repository }}, ${{ steps.deployed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} directly into the shell command string passed to wait_for_pages_deployment. An attacker controlling the deploy-repository or preview-branch input (e.g. via a PR) can inject arbitrary shell commands. Offending line: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`

Locations:

- `action.yml:177`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. The 'Generate comment content for deployment' step interpolates ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, and ${{ env.action_start_time }} directly as shell arguments to generate-comment.sh. Attacker-controlled inputs (preview-branch, deploy-repository) are expanded by the template engine before the shell sees them, enabling command injection.

Locations:

- `action.yml:187`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. The 'Wait for preview removal on GitHub Pages' step interpolates ${{ inputs.deploy-repository }}, ${{ steps.removed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} directly into the shell command string. Same injection risk as the deploy variant.

Locations:

- `action.yml:247`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. The 'Generate comment content for removal' step interpolates ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, and ${{ env.action_start_time }} directly as shell arguments. Same injection risk as the deploy variant.

Locations:

- `action.yml:257`

### github-env-injection (severity: high)

lib/main.sh writes values derived from user-controlled inputs to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The env vars $deployment_action (from inputs.action), $preview_file_path (derived from inputs.umbrella-dir and inputs.pr-number/github.event.number), $pages_base_url (from inputs.pages-base-url), $preview_url_path, $preview_url, and $action_repository (from github.action_repository/github.repository) are all written directly. A newline in any of these values allows an attacker to inject arbitrary environment variable assignments into $GITHUB_ENV, or poison $GITHUB_OUTPUT with forged key=value pairs.

Locations:

- `lib/main.sh:43`
- `lib/main.sh:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:191`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:193`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:261`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:263`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all 15 findings across action.yml and lib/main.sh:

1. action.yml - 'Wait for preview deployment on GitHub Pages' (deploy): Added env: block with WAIT_DEPLOY_REPO, WAIT_DEPLOYED_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN; replaced ${{ }} expressions in run: with $VAR references.

2. action.yml - 'Generate comment content for deployment': Added env: block with COMMENT_PREVIEW_BRANCH, COMMENT_DEPLOY_REPO, COMMENT_SERVER_URL; replaced ${{ inputs.* }} and ${{ github.server_url }} in run: with $VAR references. The ${{ env.* }} values were already available as plain env vars from the setup step.

3. action.yml - 'Wait for preview removal on GitHub Pages' (remove): Same fix as deploy variant.

4. action.yml - 'Generate comment content for removal': Same fix as deploy variant.

5. lib/main.sh - Added sanitization of all user-controlled values before writing to $GITHUB_ENV and $GITHUB_OUTPUT using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newlines that could allow injection of arbitrary env var assignments.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 183) and 'Generate comment content for removal' (line 238) steps in action.yml. Both steps used a static 'EOF' heredoc delimiter when writing multi-line $CONTENT to $GITHUB_OUTPUT, which could be exploited by attacker-controlled inputs (preview-branch, deploy-repository) containing a line literally equal to 'EOF'. The fix replaces the static delimiter with a cryptographically random 32-hex-character string generated at runtime via `DELIM="$(openssl rand -hex 16)"`, making it impossible for attacker-controlled content to match the delimiter. The $GITHUB_OUTPUT redirect was also properly quoted.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 218) and 'Generate comment content for removal' (line 300) steps in action.yml. Each untrusted input value (action_repository, action_version, preview_url, COMMENT_PREVIEW_BRANCH, COMMENT_SERVER_URL, COMMENT_DEPLOY_REPO, action_start_time) is now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being passed to generate-comment.sh. This prevents malicious callers from injecting newlines into $GITHUB_OUTPUT via these values. The sanitized values are stored in safe_* prefixed variables and passed to the script instead of the raw values.

