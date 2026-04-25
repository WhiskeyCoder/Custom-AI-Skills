# Gitea Integration Skill

## Overview

This skill provides agent-native access to a Gitea instance (v1.26.0) for code repository management, source code inspection, and issue tracking.

Gitea serves as a self-hosted source control platform for managing repositories, collaborating on code, and maintaining development workflows.

---

## Gitea Instance

* **URL:** `http://<HOST>:<PORT>`
* **API Base:** `http://<HOST>:<PORT>/api/v1`
* **Version:** `1.26.0`
* **Authentication:** Required for all mutating operations and private repository access
* **Swagger Docs:** `http://<HOST>:<PORT>/api/v1/swagger`

---

## Authentication

Two authentication methods are supported:

### Method 1: Basic Auth (Local / Self-hosted)

```bash
# Replace with your credentials
USERNAME="<YOUR_USERNAME>"
PASSWORD="<YOUR_PASSWORD>"

AUTH_HEADER="Authorization: Basic $(echo -n "$USERNAME:$PASSWORD" | base64)"

# Example:
curl -u "$USERNAME:$PASSWORD" "http://<HOST>:<PORT>/api/v1/user"
```

---

### Method 2: Token Auth

```bash
# Generate token:
# POST /api/v1/users/{username}/tokens
# Body: {"name":"token-name","scopes":["all"]}

AUTH_HEADER="Authorization: token <YOUR_GITEA_API_TOKEN>"
```

**Note (Gitea 1.26):**

* Token creation via API returns only a partial hash
* Full token is not retrievable after creation
* Basic Auth is often required for continued operations

**Valid scopes:**

* `all`
* `read:activitypub`
* `read:issue`
* `write:misc`
* `read:notification`
* `read:organization`
* `read:package`
* `read:repository`
* `read:user`

---

## Repository Operations

### List Repositories

```bash
# Public repos
curl -s "http://<HOST>:<PORT>/api/v1/repos/search?type=source&limit=20" \
  -H "Authorization: token $GITEA_TOKEN"

# User repos
curl -s "http://<HOST>:<PORT>/api/v1/user/repos?type=owner&limit=50" \
  -H "Authorization: token $GITEA_TOKEN"
```

---

### Get Repository Info

```bash
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}" \
  -H "Authorization: token $GITEA_TOKEN"
```

---

### Create Repository (Migration)

```bash
curl -s -X POST "http://<HOST>:<PORT>/api/v1/repos/migrate" \
  -H "Authorization: token $GITEA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clone_addr": "https://github.com/example/repo.git",
    "uid": 1,
    "repo_name": "new-repo",
    "repo_owner": "<OWNER>",
    "private": true,
    "auto_init": true
  }'
```

---

## Code & File Operations

### List Repository Files

```bash
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/contents?ref=main" \
  -H "Authorization: token $GITEA_TOKEN"
```

---

### Get File Content

```bash
curl -s -u "$USERNAME:$PASSWORD" \
"http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/contents/{filepath}?ref=main" \
| python3 -c "
import sys, json, base64
d = json.load(sys.stdin)
content = base64.b64decode(d['content']).decode('utf-8', errors='replace')
print(content[:2000])
"
```

---

### Get Raw File

```bash
curl -s "http://<HOST>:<PORT>/{owner}/{repo}/raw/branch/main/{filepath}"
```

---

### Create / Update File

```bash
curl -s -u "$USERNAME:$PASSWORD" \
-X PUT "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/contents/{filepath}" \
-H "Content-Type: application/json" \
-d '{
  "branch": "main",
  "sha": "current_file_sha",
  "content": "base64-encoded-content",
  "message": "Update message"
}'
```

**Notes:**

* Content must be base64 encoded
* Use `sha` from GET `/contents` for updates

---

## Branch & Tag Management

### List Branches

```bash
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/branches" \
  -H "Authorization: token $GITEA_TOKEN"
```

---

### Create Branch

```bash
curl -s -u "$USERNAME:$PASSWORD" -X POST \
"http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/branches" \
-H "Content-Type: application/json" \
-d '{
  "old_branch_name": "main",
  "new_branch_name": "feature/example"
}'
```

---

## Pull Requests

### List PRs

```bash
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/pulls?state=open" \
  -H "Authorization: token $GITEA_TOKEN"
```

---

### Create PR

```bash
curl -s -X POST "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/pulls" \
  -H "Authorization: token $GITEA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Feature update",
    "body": "Description of changes",
    "head": "feature-branch",
    "base": "main"
  }'
```

---

## Issues

### Create Issue

```bash
curl -s -X POST "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/issues" \
  -H "Authorization: token $GITEA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Issue title",
    "body": "Issue description"
  }'
```

---

## Search

```bash
# Repositories
curl -s "http://<HOST>:<PORT>/api/v1/repos/search?q=keyword"

# Issues
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/issues?q=keyword"

# Code
curl -s "http://<HOST>:<PORT>/api/v1/repos/search/code?q=keyword"
```

---

## Commits

```bash
# List commits
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/commits"

# Get commit
curl -s "http://<HOST>:<PORT>/api/v1/repos/{owner}/{repo}/commits/{sha}"
```

---

## Error Handling

| Code | Meaning                                |
| ---- | -------------------------------------- |
| 401  | Unauthorized – invalid or missing auth |
| 403  | Forbidden – insufficient permissions   |
| 404  | Not found – invalid path or resource   |
| 422  | Validation error – bad request body    |
| 429  | Rate limited – retry after reset       |
| 500  | Server error – retry with backoff      |

---

## Rate Limiting

Headers returned:

* `X-RateLimit-Limit`
* `X-RateLimit-Remaining`
* `X-RateLimit-Reset`

If remaining hits **0**, wait until reset timestamp before retrying.

---

## Best Practices

* Use pull requests for all changes (avoid direct commits to main)
* Protect main branches with review requirements
* Treat tokens and credentials as secrets
* Validate API responses and handle errors gracefully
* Use commit history for traceability and auditing
* Ensure file updates use correct `sha` values to avoid conflicts

---

## Notes for AI / Agent Usage

* Prefer read operations via token authentication
* Use Basic Auth when required for local instances
* Always validate repository paths and branches before operations
* When modifying files:

  * Fetch latest `sha`
  * Encode content correctly
  * Provide meaningful commit messages
* Monitor rate limits for large-scale automation tasks

---
