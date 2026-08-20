<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.1** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in action.yml directly interpolate GitHub Actions expressions inside shell command strings. The 'Wait for preview deployment on GitHub Pages' step calls: wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}". The 'Wait for preview removal on GitHub Pages' step has the same pattern with ${{ steps.removed-commit.outputs.deployed_commit_sha }}. These ${{ }} expressions are expanded by the Actions template engine before the shell sees the string, allowing an attacker-controlled value (e.g. inputs.deploy-repository or inputs.preview-branch) to inject arbitrary shell commands.

Locations:

- `action.yml:175`
- `action.yml:240`

### github-env-injection (severity: high)

lib/main.sh writes multiple user-controlled values to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization (no 'printf "%s" | tr -d "\n\r"' step). The variables umbrella_path, pr_number, pages_base_url, pages_base_path, deployment_repository, and deprecated_custom_url are all sourced from inputs.* and github.* context (set via the env: block in action.yml) and are written directly with echo into $GITHUB_ENV and $GITHUB_OUTPUT. A newline in any of these values could inject additional environment variable assignments or output parameters.

Locations:

- `lib/main.sh:40`
- `lib/main.sh:52`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag instead of a pinned 40-character commit SHA, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: deploy-example.yml: 'actions/checkout@v4' and 'JamesIves/github-pages-deploy-action@v4'; preview-example.yml: 'actions/checkout@v4' and 'rossjrw/pr-preview-action@v1'; test.yml: 'actions/checkout@v4' (5 occurrences across all jobs).

Locations:

- `.github/workflows/deploy-example.yml:9`
- `.github/workflows/deploy-example.yml:12`
- `.github/workflows/preview-example.yml:13`
- `.github/workflows/preview-example.yml:16`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:34`
- `.github/workflows/test.yml:98`
- `.github/workflows/test.yml:162`
- `.github/workflows/test.yml:248`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level 'permissions:' key, and no individual job within any of these files defines a 'permissions:' key either. Without explicit permissions, workflows run with the default (potentially write) token permissions, violating the principle of least privilege. Affected files: test.yml, deploy-example.yml, preview-example.yml.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/deploy-example.yml:1`
- `.github/workflows/preview-example.yml:1`

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

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings:

1. script-injection / static-inline-injection (action.yml): Moved all ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, ${{ inputs.token }}, and ${{ steps.*.outputs.deployed_commit_sha }} expressions from run: blocks into env: blocks for both 'Wait for preview deployment on GitHub Pages' and 'Wait for preview removal on GitHub Pages' steps. Shell commands now reference plain env vars ($WAIT_DEPLOY_REPO, $WAIT_DEPLOYED_SHA, $WAIT_PREVIEW_BRANCH, $WAIT_TOKEN).

2. github-env-injection (lib/main.sh): Added sanitization of all user-controlled values using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT. Sanitized variables: umbrella_path, pr_number, pages_base_url, pages_base_path, deployment_repository, preview_file_path, preview_url_path, action_version, action_start_time, github_action_repository.

3. unpinned-uses: Pinned all mutable tag references to full commit SHAs in deploy-example.yml, preview-example.yml, and test.yml:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - JamesIves/github-pages-deploy-action@v4 → @fa24774553152dd7873cd16ebd8d959b010c5445 # v4
   - rossjrw/pr-preview-action@v1 → @ffa7509e91a3ec8dfc2e5536c4d5c1acdf7a6de9 # v1

4. missing-permissions: Added top-level permissions blocks to all three workflow files with minimal required permissions.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings:

1. **github-env-injection (lib/main.sh)**: Added `safe_deployment_action=$(printf '%s' "$deployment_action" | tr -d '\n\r')` and replaced both raw `echo "deployment_action=$deployment_action"` occurrences (GITHUB_ENV at line 57 and GITHUB_OUTPUT at line 67) with the sanitized `safe_deployment_action` variable.

2. **script-injection (.github/workflows/test.yml)**: Fixed all `run:` blocks with direct `${{ }}` expression interpolation:
   - Modify fixture steps: replaced `${{ env.TEST_ID }}` in sed commands with shell env var `$TEST_ID` (already available from job-level env) using proper shell quoting.
   - Verify deployed/redeployed steps: moved `${{ env.TEST_ID }}-INITIALISED` and `${{ env.TEST_ID }}-REDEPLOYED` into `env: EXPECTED_CONTENT:` blocks, referenced as `"$EXPECTED_CONTENT"` in run.
   - Cleanup steps (all 4 jobs): added `env: DEPLOY_TOKEN: ${{ secrets.TEST_DEPLOY_TOKEN }}` and replaced `"${{ secrets.TEST_DEPLOY_TOKEN }}"` positional arguments with `"$DEPLOY_TOKEN"`. Other variables (TEST_DEPLOY_REPO, TEST_ID, TEST_UMBRELLA_DIR, CUSTOM_UMBRELLA_DIR) were already available as shell env vars from job-level env blocks.

