<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.3** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings, enabling script injection. (1) The 'Wait for preview deployment on GitHub Pages' step passes `${{ inputs.deploy-repository }}`, `${{ steps.deployed-commit.outputs.deployed_commit_sha }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` as unquoted shell arguments directly in the run block. (2) The 'Generate comment content for deployment' step interpolates `${{ env.action_repository }}`, `${{ env.action_version }}`, `${{ env.preview_url }}`, `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, `${{ inputs.deploy-repository }}`, and `${{ env.action_start_time }}` directly in the shell. (3) The 'Wait for preview removal on GitHub Pages' step has the same pattern as (1) with `${{ steps.removed-commit.outputs.deployed_commit_sha }}`. (4) The 'Generate comment content for removal' step has the same pattern as (2). An attacker controlling a PR can inject arbitrary shell commands via these inputs.

Locations:

- `action.yml:168`
- `action.yml:181`
- `action.yml:233`
- `action.yml:246`

### github-env-injection (severity: high)

Multiple unsanitized writes of untrusted input-derived values to $GITHUB_ENV and $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) In lib/main.sh, the env vars `$umbrella_path`, `$pr_number`, `$pages_base_url`, `$pages_base_path`, `$deployment_repository`, `$github_action_ref`, `$github_action_repository` are all set from `inputs.*` and `github.*` context values in the calling step's `env:` block, then written directly to `$GITHUB_ENV` (lines ~43-51) and `$GITHUB_OUTPUT` (lines ~54-62) without sanitization. A newline in any of these values can inject arbitrary environment variables. (2) In the 'Generate comment content for deployment' and 'Generate comment content for removal' run blocks in action.yml, `$CONTENT` is derived from `${{ inputs.* }}` values passed as shell arguments and then written to `$GITHUB_OUTPUT` via a heredoc without sanitization.

Locations:

- `lib/main.sh:43`
- `lib/main.sh:54`
- `action.yml:192`
- `action.yml:257`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script injection and github-env-injection findings:

1. action.yml - 'Wait for preview deployment on GitHub Pages': Moved ${{ inputs.deploy-repository }}, ${{ steps.deployed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, ${{ inputs.token }} from the run: block into an env: block as WAIT_DEPLOY_REPO, WAIT_DEPLOYED_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN.

2. action.yml - 'Generate comment content for deployment': Moved ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }} from the run: block into an env: block as GC_PREVIEW_BRANCH, GC_SERVER_URL, GC_DEPLOY_REPO. The env.* values (action_repository, action_version, preview_url, action_start_time) are already available as environment variables from the sanitized main.sh step.

3. action.yml - 'Wait for preview removal on GitHub Pages': Same fix as the deploy wait step, using WAIT_DEPLOY_REPO, WAIT_REMOVED_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN.

4. action.yml - 'Generate comment content for removal': Same fix as the deploy comment step.

5. lib/main.sh - Added a sanitize() helper function using printf '%s' "$1" | tr -d '\n\r' and applied it to all values before writing to $GITHUB_ENV and $GITHUB_OUTPUT, preventing newline-based environment variable injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection, hardcoded-credentials, missing-permissions, script-injection, unpinned-uses

**Notes:**

Fixed all 5 findings:

1. github-env-injection (action.yml): Both comment-generation steps now sanitize $CONTENT with `printf '%s' "$CONTENT" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT as a single-line value, preventing heredoc injection.

2. hardcoded-credentials (test.yml): Replaced `token: fake-token` in both test steps with `token: ${{ secrets.GITHUB_TOKEN }}`.

3. missing-permissions: Added top-level `permissions:` blocks to all 4 workflow files: deploy-example.yml (contents: write), preview-example.yml (contents: write, pull-requests: write), test.yml (contents: read), test-integration.yml (contents: read).

4. script-injection (test-integration.yml): Moved all ${{ }} expressions from run: blocks to env: blocks in all affected steps — Modify fixture steps (sed commands now use $TEST_ID env var), Verify steps (test script arguments now use env vars), and all Cleanup steps (cleanup_deployment arguments now use CLEANUP_* env vars).

5. unpinned-uses: Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in all 4 workflow files, and JamesIves/github-pages-deploy-action@v4 to SHA fa24774553152dd7873cd16ebd8d959b010c5445 in deploy-example.yml.

