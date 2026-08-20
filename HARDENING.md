<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 13 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four run: blocks in action.yml directly interpolate ${{ }} expressions into shell command strings, allowing script injection. (1) 'Wait for preview deployment on GitHub Pages' (deploy): `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"` — attacker-controlled inputs are interpolated directly into the shell before quoting. (2) 'Wait for preview removal on GitHub Pages' (remove): same pattern with `${{ inputs.deploy-repository }}`, `${{ steps.removed-commit.outputs.deployed_commit_sha }}`, `${{ inputs.preview-branch }}`, `${{ inputs.token }}`. (3) 'Generate comment content for deployment': `$GITHUB_ACTION_PATH/lib/generate-comment.sh` is called with `"${{ env.action_repository }}"`, `"${{ env.action_version }}"`, `"${{ env.preview_url }}"`, `"${{ inputs.preview-branch }}"`, `"${{ github.server_url }}"`, `"${{ inputs.deploy-repository }}"`, `"${{ env.action_start_time }}"`, `"${{ inputs.qr-code }}"` interpolated directly into the run: shell string. (4) 'Generate comment content for removal': same pattern. Any of these inputs could contain shell metacharacters injected by an attacker via a pull request or workflow_dispatch event.

Locations:

- `action.yml:176`
- `action.yml:195`
- `action.yml:207`
- `action.yml:260`

### github-env-injection (severity: high)

lib/main.sh writes values derived from caller-controlled environment variables directly to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The setup step's env: block maps inputs.* and github.* context values into shell env vars (umbrella_path, pr_number, pages_base_url, pages_base_path, deployment_repository, action_repository, action_ref, deprecated_custom_url, deployment_action). These are then used to construct values like preview_file_path, preview_url_path, preview_url, and action_version, all of which are written unsanitized to $GITHUB_ENV (lines ~42-51) and $GITHUB_OUTPUT (lines ~54-62). An attacker controlling inputs such as inputs.umbrella-dir, inputs.pr-number, inputs.pages-base-url, inputs.deploy-repository, or inputs.custom-url could inject newlines to set arbitrary environment variables or outputs.

Locations:

- `lib/main.sh:42`
- `lib/main.sh:54`

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

Fixed all script injection findings in action.yml by moving ${{ }} expressions from run: blocks into env: blocks for four steps: 'Wait for preview deployment on GitHub Pages', 'Generate comment content for deployment', 'Wait for preview removal on GitHub Pages', and 'Generate comment content for removal'. Fixed github-env-injection in lib/main.sh by sanitizing all user-controlled values with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across four workflow files:

1. unpinned-uses: Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in all four workflow files. Pinned JamesIves/github-pages-deploy-action@v4 to SHA fa24774553152dd7873cd16ebd8d959b010c5445 in deploy-example.yml.

2. missing-permissions: Added top-level permissions blocks to all four files: deploy-example.yml gets 'contents: write' (for gh-pages deployment), preview-example.yml gets 'contents: write' + 'pull-requests: write' (for PR preview), test-integration.yml and test.yml get 'contents: read' (minimal for checkout).

3. script-injection: In test-integration.yml, all ${{ }} expressions in run: blocks were moved to step-level env: blocks. 'Modify fixture' steps use STEP_TEST_ID env var instead of ${{ env.TEST_ID }} directly in sed commands. 'Verify' steps use STEP_TEST_ID env var for the shell argument. All 'Cleanup' steps use CLEANUP_DEPLOY_REPO, CLEANUP_TEST_ID, CLEANUP_TOKEN, CLEANUP_UMBRELLA_DIR, and CLEANUP_CUSTOM_UMBRELLA_DIR env vars instead of inline ${{ }} expressions.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 222) and 'Generate comment content for removal' (line 288) steps in action.yml. Replaced the fixed 'EOF' heredoc delimiter with a randomly generated hex string via `DELIM=$(openssl rand -hex 16)` and used `${DELIM}` as the delimiter. This prevents injection attacks where user-controlled inputs (preview-branch, deploy-repository, qr-code, server_url) could contain a line exactly matching 'EOF' to prematurely terminate the delimiter and inject arbitrary key=value pairs into $GITHUB_OUTPUT. Also quoted `$GITHUB_OUTPUT` for correctness.

