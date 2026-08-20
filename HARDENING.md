<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.2** was hardened automatically. 12 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell commands, enabling script injection. (1) The 'Wait for preview deployment on GitHub Pages' step passes `${{ inputs.deploy-repository }}`, `${{ inputs.preview-branch }}`, `${{ inputs.token }}`, and `${{ steps.deployed-commit.outputs.deployed_commit_sha }}` as shell arguments directly. (2) The 'Generate comment content for deployment' step interpolates `${{ env.action_repository }}`, `${{ env.action_version }}`, `${{ env.preview_url }}`, `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, `${{ inputs.deploy-repository }}`, and `${{ env.action_start_time }}` directly in the shell. (3) The 'Wait for preview removal on GitHub Pages' step has the same pattern as (1). (4) The 'Generate comment content for removal' step has the same pattern as (2). An attacker who controls any of these inputs (e.g. via a malicious PR or workflow_dispatch) can inject arbitrary shell commands.

Locations:

- `action.yml:170`
- `action.yml:185`
- `action.yml:240`
- `action.yml:255`

### github-env-injection (severity: high)

lib/main.sh writes multiple values derived from caller-controlled inputs to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). The `env:` block in the 'Setup preview environment' step maps `inputs.umbrella-dir` → `$umbrella_path`, `inputs.pr-number` → `$pr_number`, `inputs.deploy-repository` → `$deployment_repository`, `inputs.pages-base-url` → `$pages_base_url`, `inputs.pages-base-path` → `$pages_base_path`, and `inputs.action` → `$deployment_action`. These are then used to compute `preview_file_path`, `preview_url_path`, `preview_url`, `pages_base_url`, `deployment_action`, `action_repository`, and `action_start_time`, all of which are written unsanitized to `$GITHUB_ENV` (lines 43–52) and `$GITHUB_OUTPUT` (lines 55–63). A newline in any input value can inject arbitrary environment variables or output parameters.

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

Fixed all 12 static-inline-injection/script-injection findings in action.yml by moving every ${{ }} expression from run: shell strings into step-level env: blocks, then referencing them as plain shell variables. Fixed the github-env-injection finding in lib/main.sh by adding a sanitize() helper (printf '%s' "$1" | tr -d '\n\r') and applying it to all values derived from caller-controlled inputs before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** unpinned-uses, permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across three workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262` and `JamesIves/github-pages-deploy-action@v4` to SHA `fa24774553152dd7873cd16ebd8d959b010c5445` in all workflow files.

2. **permissions**: Added top-level `permissions: {}` to all three workflow files (test.yml, deploy-example.yml, preview-example.yml). Added job-level `permissions: contents: write` for deploy jobs and `contents: write` + `pull-requests: write` for the preview job.

3. **script-injection**: Moved all `${{ env.TEST_ID }}`, `${{ env.TEST_DEPLOY_REPO }}`, `${{ secrets.TEST_DEPLOY_TOKEN }}`, `${{ env.TEST_UMBRELLA_DIR }}`, and `${{ env.CUSTOM_UMBRELLA_DIR }}` expressions out of `run:` blocks into step-level `env:` blocks. Shell scripts now reference these as plain environment variables (e.g., `$CLEANUP_TOKEN`, `$EXPECTED_CONTENT`). The sed commands in 'Modify fixture' steps use shell variable expansion via `'"$TEST_ID"'` quoting pattern to avoid template injection.

4. **hardcoded-credentials**: Replaced `token: fake-token` with `token: ${{ secrets.COMMENT_TEST_TOKEN }}` in both steps of the test-comment-content job.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 163) and 'Generate comment content for removal' (line 240) steps in action.yml. Both steps used a fixed 'EOF' heredoc delimiter when writing multi-line CONTENT to $GITHUB_OUTPUT, which allowed injection via user-controlled inputs (preview-branch, deploy-repository). The fix replaces the static 'EOF' delimiter with a randomly generated 32-hex-character string via `openssl rand -hex 16`, making delimiter prediction cryptographically infeasible. Also changed `echo "$CONTENT"` to `printf '%s\n' "$CONTENT"` for safer output and properly quoted `"$GITHUB_OUTPUT"`.

