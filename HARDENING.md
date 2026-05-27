# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Four `run:` blocks in action.yml directly interpolate `inputs.*` and `github.*` expressions into shell command strings without first assigning them to environment variables. This allows an attacker to inject arbitrary shell commands via crafted input values.

1. 'Wait for preview deployment on GitHub Pages' step: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" ... "${{ inputs.preview-branch }}" "${{ inputs.token }}"` — inputs interpolated directly as shell arguments.
2. 'Generate comment content for deployment' step: `"${{ inputs.preview-branch }}"`, `"${{ github.server_url }}"`, `"${{ inputs.deploy-repository }}"`, `"${{ inputs.qr-code }}"` — all interpolated directly as shell arguments to generate-comment.sh.
3. 'Wait for preview removal on GitHub Pages' step: same pattern as #1.
4. 'Generate comment content for removal' step: same pattern as #2.

All these inputs should be assigned to environment variables via `env:` and referenced as `$ENV_VAR` in the run block.

Locations:

- `action.yml:162`
- `action.yml:172`
- `action.yml:228`
- `action.yml:242`

### github-env-injection (severity: high)

lib/main.sh (invoked by the 'Setup preview environment' step in action.yml) receives attacker-controlled values from `inputs.*` via `env:` variables — including `umbrella_path` (inputs.umbrella-dir), `pr_number` (inputs.pr-number), `pages_base_url` (inputs.pages-base-url), `pages_base_path` (inputs.pages-base-path), `deployment_action` (inputs.action), `deployment_repository` (inputs.deploy-repository), and `deprecated_custom_url` (inputs.custom-url) — and writes them (or values derived from them) directly to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A malicious input containing newlines could inject arbitrary key=value pairs into the GitHub environment, leading to environment variable injection in subsequent steps.

Locations:

- `lib/main.sh:40`
- `lib/main.sh:52`

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

Fixed all script injection findings in action.yml by moving ${{ inputs.* }} and ${{ github.* }} expressions from run: shell strings into env: blocks, then referencing them as plain environment variables. Fixed github-env-injection in lib/main.sh by adding a sanitize() helper function that strips newlines (tr -d '\n\r') and applying it to all attacker-controlled values before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 175) and 'Generate comment content for removal' (line 249) steps in action.yml:

1. Sanitized attacker-controlled inputs (PREVIEW_BRANCH, DEPLOY_REPOSITORY, QR_CODE) using `printf '%s' "$VAR" | tr -d '\n\r'` before passing them to generate-comment.sh.

2. Replaced the static 'EOF' heredoc delimiter with a randomly generated unique delimiter (`EOF_$(openssl rand -hex 16)`) to prevent an attacker from injecting the literal 'EOF' string to terminate the heredoc and poison $GITHUB_OUTPUT.

3. Quoted `"$GITHUB_OUTPUT"` for correctness.

