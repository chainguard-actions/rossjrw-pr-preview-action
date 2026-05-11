# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 16 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Wait for preview deployment on GitHub Pages' (deploy) run: block directly interpolates attacker-controlled expressions into shell command arguments without first assigning them to environment variables: `${{ inputs.deploy-repository }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` are passed directly as arguments to `wait_for_pages_deployment`. An attacker controlling these inputs could inject arbitrary shell commands.

Locations:

- `action.yml:192`

### script-injection (severity: high)

The 'Generate comment content for deployment' run: block directly interpolates attacker-controlled expressions into shell command arguments: `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, `${{ inputs.deploy-repository }}`, and `${{ inputs.qr-code }}` are passed directly as arguments to generate-comment.sh without being assigned to environment variables first.

Locations:

- `action.yml:204`

### script-injection (severity: high)

The 'Wait for preview removal on GitHub Pages' run: block directly interpolates attacker-controlled expressions into shell command arguments without first assigning them to environment variables: `${{ inputs.deploy-repository }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` are passed directly as arguments to `wait_for_pages_deployment`.

Locations:

- `action.yml:244`

### script-injection (severity: high)

The 'Generate comment content for removal' run: block directly interpolates attacker-controlled expressions into shell command arguments: `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, and `${{ inputs.deploy-repository }}` are passed directly as arguments to generate-comment.sh without being assigned to environment variables first.

Locations:

- `action.yml:256`

### github-env-injection (severity: high)

lib/main.sh writes values derived from attacker-controlled inputs to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Variables written include: `preview_file_path` (derived from `$umbrella_path` = inputs.umbrella-dir and `$pr_number` = inputs.pr-number/github.event.number), `pages_base_url` (from inputs.pages-base-url), `preview_url_path`, `preview_url`, `action_repository` (from github.action_repository/github.repository), and `deployment_action` (from inputs.action). A newline injection in any of these inputs could allow an attacker to set arbitrary environment variables or outputs.

Locations:

- `lib/main.sh:40`
- `lib/main.sh:51`

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

Fixed all 16 findings across action.yml and lib/main.sh:

1. action.yml - 'Wait for preview deployment on GitHub Pages' (deploy): Moved inputs.deploy-repository, inputs.preview-branch, and inputs.token into an env: block (DEPLOY_REPOSITORY, PREVIEW_BRANCH, DEPLOY_TOKEN); replaced ${{ }} interpolations in run: with plain $VAR references.

2. action.yml - 'Generate comment content for deployment': Moved inputs.preview-branch, github.server_url, inputs.deploy-repository, and inputs.qr-code into an env: block (PREVIEW_BRANCH, SERVER_URL, DEPLOY_REPOSITORY, QR_CODE); replaced ${{ }} interpolations in run: with plain $VAR references.

3. action.yml - 'Wait for preview removal on GitHub Pages': Moved inputs.deploy-repository, inputs.preview-branch, and inputs.token into an env: block (DEPLOY_REPOSITORY, PREVIEW_BRANCH, DEPLOY_TOKEN); replaced ${{ }} interpolations in run: with plain $VAR references.

4. action.yml - 'Generate comment content for removal': Moved inputs.preview-branch, github.server_url, and inputs.deploy-repository into an env: block (PREVIEW_BRANCH, SERVER_URL, DEPLOY_REPOSITORY); replaced ${{ }} interpolations in run: with plain $VAR references.

5. lib/main.sh - Added a safe() helper function using printf '%s' "$1" | tr -d '\n\r' and wrapped all attacker-controlled values written to $GITHUB_ENV and $GITHUB_OUTPUT with safe() calls to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 201) and 'Generate comment content for removal' (line 284) steps in action.yml. For each step: (1) sanitized attacker-controlled input env vars (PREVIEW_BRANCH, SERVER_URL, DEPLOY_REPOSITORY, QR_CODE) using `printf '%s' "$VAR" | tr -d '\n\r'` before passing them to generate-comment.sh; (2) replaced the static 'EOF' heredoc delimiter with a random `EOF_$(openssl rand -hex 16)` to prevent delimiter injection attacks; (3) quoted `$GITHUB_OUTPUT` properly.

