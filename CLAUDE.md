# Titan Banking Organization-Wide Claude Code Instructions

This file contains common workflows, conventions, and best practices for all Titan Banking repositories.

**Public URL:** https://raw.githubusercontent.com/Titan-banking/claude/refs/heads/main/CLAUDE.md

## Git Commit Message Convention

**IMPORTANT**: All commit messages MUST follow the Conventional Commits specification.

### Format
```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Commit Types
- **feat**: A new feature for the user
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

### Examples
```bash
feat(detection): add support for custom PII patterns
fix(gliner): resolve model loading timeout
docs(api): add OpenAPI examples for validation endpoint
refactor(models): optimize model caching strategy
perf(inference): reduce inference latency by 30%
test(validation): add comprehensive PII detection tests
ci(docker): optimize image build size
```

### Guidelines
- Use lowercase for type and scope
- Keep subject line under 72 characters
- Use imperative mood ("add" not "added" or "adds")
- Don't end subject line with a period
- Separate subject from body with a blank line
- Use body to explain what and why, not how
- **Do NOT include file lists** - git tracks changes automatically
- Keep commit messages focused and concise

## Git Safety Protocol

**IMPORTANT**: Follow these safety rules when working with git:

- **NEVER force push** (`git push --force` or `git push -f`) unless the user explicitly requests it
- **NEVER force push to main/master** - warn the user if they request this
- **NEVER run destructive/irreversible git commands** (like `git reset --hard`, `git clean -fd`) unless explicitly requested
- **NEVER skip hooks** (`--no-verify`, `--no-gpg-sign`) unless the user explicitly requests it
- **NEVER update git config** without explicit user permission
- **Avoid `git commit --amend`** - only use when: (1) user explicitly requested amend, OR (2) fixing pre-commit hook changes on YOUR OWN commit
- **Before amending**: ALWAYS check authorship with `git log -1 --format='%an %ae'` to verify it's your commit
- **Prefer `git push --force-with-lease`** over `--force` when force push is explicitly requested (safer)

When resolving merge conflicts after rebase:
1. Fix the conflicts manually
2. Stage the resolved files with `git add`
3. Continue with `git rebase --continue`
4. **Ask the user** before force pushing the rebased branch

## Creating Pull Requests

**IMPORTANT**: When creating pull requests, always follow these conventions:

### Branch Naming

**IMPORTANT**: All branches must follow this naming convention:

**Format:**
```
<initials>/<JIRA-TICKET>-<short-description>
```

**Rules:**
- Use developer initials in lowercase (e.g., `sp`, `ks`, `tb`)
- JIRA ticket in uppercase (e.g., `TITAN-266`)
- Short description: 2-4 words, kebab-case, derived from either:
  - The Jira ticket title/summary
  - The actual work completed in commits
- Total branch name should be concise and scannable

**Examples:**
- `sp/TITAN-149-pii-detection-service` (from Jira title: "Extract PII detection to microservice")
- `ks/TITAN-180-tier-classification` (from Jira title: "Add PII tier classification")
- `tb/TITAN-192-sentry-integration` (from completed work: adding Sentry error tracking)

**Creating Branch Names:**
1. Extract key concept from Jira title/description
2. OR summarize the actual work completed in 2-4 words
3. Use kebab-case (lowercase with hyphens)
4. Keep it meaningful but concise

### Pull Request Title Format
```
JIRA-TICKET: One sentence summary of changes
```

**Examples:**
- `TITAN-149: Extract PII detection into standalone microservice`
- `TITAN-180: Add PII tier classification system`
- `TITAN-192: Integrate Sentry for error tracking`

**Guidelines:**
- Always extract and include the Jira ticket from the branch name (uppercase)
- Title should be one concise sentence summarizing the main change
- Use imperative mood ("Add" not "Added", "Fix" not "Fixed")
- Keep title under 72 characters when possible

### Pull Request Description Format

**Keep PR descriptions concise and focused:**

```markdown
## Summary
{1-3 sentence summary of what was done and why}

## Key Features
- {Feature 1}
- {Feature 2}
- {Feature 3}

## Benefits
- {Benefit 1}
- {Benefit 2}
- {Benefit 3}

## Testing
- {Testing approach and validation performed}
- {How to test/verify the changes}
- {Test command examples if applicable}

## Concerns / Alternatives
{Optional: Any concerns, tradeoffs, or alternative approaches considered}

> Generated with [Claude Code](https://claude.com/claude-code)
```

**Important:**
- **Do NOT list all changed files** - GitHub shows this automatically
- Focus on the "what" and "why", not implementation details
- Keep descriptions scannable with bullet points

### Git Workflow for PR Creation

**IMPORTANT**: When creating pull requests, assume all code changes are already committed and pushed to the remote branch.

**Step 1: Verify Git Status**

```bash
# Check current branch name
/usr/bin/git branch --show-current

# Verify working tree is clean (everything is committed)
/usr/bin/git status

# Should see: "nothing to commit, working tree clean"
```

**Step 2: Review Recent Commits**

```bash
# View recent commits on current branch
/usr/bin/git log --oneline -5

# View commits between base branch (dev) and current branch
/usr/bin/git log origin/dev..HEAD --oneline
```

**Step 3: Analyze Changes for PR Description**

```bash
# Get file statistics between dev and current branch
/usr/bin/git diff origin/dev HEAD --stat

# This shows:
# - Which files were modified
# - Number of insertions and deletions
# - Overall scope of changes
```

**Step 4: Push to Remote (if needed)**

```bash
# Ensure branch is pushed to remote with tracking
/usr/bin/git push -u origin <branch-name>

# Should see: "Everything up-to-date" if already pushed
```

**Step 5: Extract PR Information**

Use git commands to determine:
- **Branch name**: Extract Jira ticket ID (e.g., `ks/TITAN-39` � `TITAN-39`)
- **Commit messages**: Use recent commits to understand what was done
- **File changes**: Use `git diff --stat` to see which files/areas were modified
- **Scope**: Number of files changed indicates complexity

**Best Practices:**
- Always use `/usr/bin/git` to avoid shell alias issues
- Check `git status` confirms "nothing to commit, working tree clean"
- Review `git log` to understand commit history
- Use `git diff origin/dev HEAD --stat` to see full scope of changes
- Assume user has already committed and pushed all changes

### Automatic PR Creation

When the user requests a pull request, use the `gh` CLI:

```bash
# Extract Jira ticket from branch name
BRANCH_NAME=$(git branch --show-current)
JIRA_TICKET=$(echo $BRANCH_NAME | grep -oE '[Tt][Ii][Tt][Aa][Nn]-[0-9]+' | tr '[:lower:]' '[:upper:]')

# Create PR with formatted title and description
gh pr create --title "$JIRA_TICKET: Summary of changes" --body "$(cat <<'EOF'
## Summary
Concise summary of changes

## Changes
- Major change 1
- Major change 2

## Testing
Testing instructions

> Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**Important:**
- Always include the Jira ticket in the title
- Keep title concise and descriptive
- Provide enough context in description for reviewers
- Include testing instructions

## Working with GitHub

### Querying GitHub Information

When Claude Code needs to fetch information from GitHub repositories, use the following strategies in order of preference:

#### 1. **MCP GitHub Tools (Preferred for Structured Data)**

The GitHub MCP server provides authenticated tools for accessing repository data. These work with **both public and private repositories**.

**Common Tools:**
```
mcp__github__get_pull_request         - Get PR metadata
mcp__github__get_pull_request_files   - Get list of changed files (WARNING: can exceed token limits on large PRs)
mcp__github__get_pull_request_reviews - Get PR reviews
mcp__github__get_pull_request_comments - Get PR comments
mcp__github__get_file_contents        - Get file contents from any branch
mcp__github__list_commits             - Get commit history
mcp__github__create_pull_request_review - Create PR review with comments
```

**Advantages:**
- Authenticated access to private repositories
- Structured JSON responses
- Specific tools don't require user approval (pre-approved in system prompt)
- Rich metadata and detailed information

**Limitations:**
- `get_pull_request_files` can fail on large PRs (>25,000 tokens)
- Verbose responses may consume more tokens

**Best for:** PR metadata, reviews, comments, file contents, creating reviews

#### 2. **GitHub CLI (gh) - Quick Queries**

The `gh` CLI is installed and provides fast, concise access to GitHub data.

**Common Commands:**
```bash
gh pr view <number> --repo <org>/<repo>              # View PR details
gh pr view <number> --json files --jq '.files[].path' # List changed files
gh pr diff <number> --repo <org>/<repo>               # View diff
gh pr list --repo <org>/<repo>                        # List PRs
gh issue list --repo <org>/<repo>                     # List issues
```

**Advantages:**
- Fast and concise output
- Works with private repos (uses local auth)
- Flexible JSON output with `--json` flag
- Can pipe to `jq` for filtering

**Limitations:**
- Requires parsing text/JSON output
- Less structured than MCP tools
- May need multiple commands for complex queries

**Best for:** Quick file lists, diffs, PR summaries, commit logs

#### 3. **Task Agent (For Large PRs or Complex Analysis)**

When MCP tools exceed token limits or you need cross-file analysis, delegate to a Task agent.

**Example:**
```python
Task(
  subagent_type="general-purpose",
  description="Analyze PR changes",
  prompt="""
  Compare files between branches using mcp__github__get_file_contents:
  - Base branch: dev (SHA: abc123)
  - Head branch: feature-branch (SHA: def456)

  Fetch key files and identify changes...
  """
)
```

**Strategy:**
- Use `gh` CLI to get list of changed files
- Fetch specific files from both branches via MCP tools
- Compare and summarize major modifications

**Best for:** Large PRs, architectural analysis, cross-file refactoring review

#### 4. **WebFetch (Public Repos Only - Avoid for Private Repos)**

WebFetch can access public GitHub content but **fails with 404 on private repositories**.

**Fails for private repos:**
```python
WebFetch("https://github.com/Titan-banking/modelhub/pull/17/files")  # 404 error
```

**Do NOT use for Titan-banking repositories** - they are private.

### Best Practices

1. **Start with MCP tools** for metadata: `mcp__github__get_pull_request`, `get_pull_request_reviews`, `get_pull_request_comments`
2. **Use gh CLI** for quick file lists: `gh pr view <num> --json files`
3. **Check PR size** before using `mcp__github__get_pull_request_files` (fails on large PRs)
4. **Delegate to Task agent** for complex analysis or large PRs
5. **Batch MCP calls** in parallel when fetching multiple independent pieces of data
6. **Never use WebFetch** for private Titan-banking repositories

### Example Workflow: Reviewing a PR

```python
# Step 1: Get PR metadata (MCP)
mcp__github__get_pull_request(owner="Titan-banking", repo="modelhub", pull_number=17)

# Step 2: Get reviews and comments (MCP - safe, smaller responses)
mcp__github__get_pull_request_reviews(...)
mcp__github__get_pull_request_comments(...)

# Step 3: Get file list (gh CLI - fast and concise)
gh pr view 17 --repo Titan-banking/modelhub --json files --jq '.files[].path'

# Step 4: For specific file changes (MCP)
mcp__github__get_file_contents(owner="Titan-banking", repo="modelhub",
                                 path="src/modelhub/agents/agent.py",
                                 branch="feature-branch")

# Step 5: Create review if needed (MCP)
mcp__github__create_pull_request_review(...)
```

This approach balances speed, thoroughness, and token efficiency.

## Working with JIRA

### Querying JIRA Information

The Atlassian MCP server provides authenticated tools for accessing JIRA data. All Project Titan tickets use the **TITAN** project key.

#### Common JIRA MCP Tools

**Getting Issue Details:**
```
mcp__atlassian__getJiraIssue              - Get full issue details by key or ID
mcp__atlassian__searchJiraIssuesUsingJql  - Search issues using JQL
mcp__atlassian__getTransitionsForJiraIssue - Get available status transitions
```

**Modifying Issues:**
```
mcp__atlassian__editJiraIssue             - Update issue fields (summary, description, etc.)
mcp__atlassian__transitionJiraIssue       - Change issue status (TODO, In Progress, Done)
mcp__atlassian__addCommentToJiraIssue     - Add comment to issue
```

**Creating Issues:**
```
mcp__atlassian__createJiraIssue           - Create new JIRA issue
mcp__atlassian__getVisibleJiraProjects    - List accessible projects
mcp__atlassian__getJiraProjectIssueTypesMetadata - Get issue types for project
```

#### Getting Cloud ID

Before using JIRA MCP tools, you need the Atlassian Cloud ID:

```python
# Get accessible Atlassian resources
mcp__atlassian__getAccessibleAtlassianResources()

# Returns cloud ID like: "12345678-abcd-1234-efgh-567890abcdef"
```

**Note:** You can also use the site URL directly (e.g., `"https://titan-banking.atlassian.net"`) and the tool will convert it to a Cloud ID.

#### Example: Reading a JIRA Issue

```python
# Get issue details
mcp__atlassian__getJiraIssue(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    issueIdOrKey="TITAN-149"
)

# Returns:
# - Summary, description, status, assignee
# - Issue type, priority, labels
# - Comments, attachments, links
# - Custom fields
```

#### Example: Updating JIRA Issue Description

```python
# Update issue description (supports Markdown)
mcp__atlassian__editJiraIssue(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    issueIdOrKey="TITAN-149",
    fields={
        "description": """
# Summary
Extracted PII detection functionality from modelhub into standalone microservice.

## Problem
Modelhub backend was consuming ~3GB RAM and taking 2-3 minutes to start due to ML model loading.

## Solution
Created new pii-api microservice with single /validate/pii endpoint...
        """
    }
)
```

#### Example: Searching for Issues

```python
# Search using JQL (JIRA Query Language)
mcp__atlassian__searchJiraIssuesUsingJql(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    jql="project = TITAN AND status = 'In Progress' ORDER BY updated DESC",
    maxResults=50
)

# Common JQL queries:
# - "project = TITAN AND assignee = currentUser()"
# - "project = TITAN AND status = 'To Do' AND priority = High"
# - "project = TITAN AND text ~ 'PII detection'"
```

#### Example: Transitioning Issue Status

```python
# Step 1: Get available transitions
transitions = mcp__atlassian__getTransitionsForJiraIssue(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    issueIdOrKey="TITAN-149"
)

# Step 2: Transition to new status
mcp__atlassian__transitionJiraIssue(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    issueIdOrKey="TITAN-149",
    transition={"id": "31"}  # ID from transitions list
)
```

#### Example: Adding Comments

```python
# Add comment to issue (supports Markdown)
mcp__atlassian__addCommentToJiraIssue(
    cloudId="12345678-abcd-1234-efgh-567890abcdef",
    issueIdOrKey="TITAN-149",
    commentBody="Pull requests created:\n- modelhub #43\n- infra #26\n- pii-api #1"
)
```

### JIRA Best Practices

1. **Always use ticket keys** - Reference JIRA tickets by key (e.g., `TITAN-149`) not by ID
2. **Update descriptions comprehensively** - Include problem statement, solution, technical details, and PR links
3. **Use Markdown formatting** - JIRA supports Markdown in descriptions and comments
4. **Link PRs to tickets** - Always mention PR numbers and links in ticket comments
5. **Keep tickets updated** - Transition status as work progresses (To Do � In Progress � Done)
6. **Search before creating** - Use JQL to check if similar issues exist

### JIRA + GitHub Workflow

When completing a feature:

1. **Create branch** with JIRA ticket: `sp/TITAN-149-feature-name`
2. **Implement changes** across repositories
3. **Create PRs** with JIRA ticket in title: `TITAN-149: Summary of changes`
4. **Update JIRA ticket** with comprehensive description and PR links
5. **Add comment** with PR URLs for easy reference
6. **Transition status** to "In Review" or "Done"
