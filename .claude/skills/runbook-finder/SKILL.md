# Runbook Finder Skill

Find and retrieve runbooks from the orbit-runbooks GitHub repository for incident triage.

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `RUNBOOK_REPO` | GitHub repository with runbooks | `techops-services/orbit-runbooks` |
| `RUNBOOK_BRANCH` | Branch to fetch runbooks from | `main` |

## Usage

Invoke this skill when you need to find the appropriate runbook for an alert.

## Input

Provide the alert JSON or the `alert_fields.runbook` URL from the alert.

## Instructions

1. **Primary: Extract runbook URL from alert**
   - Look for `alert_fields.runbook` field in the alert JSON
   - Extract the filename from the URL (e.g., `disk-pressure.md` from `https://github.com/techops-services/orbit-runbooks/blob/main/disk-pressure.md`)

2. **Fallback: Fuzzy match alert to runbook**
   If no runbook URL is provided, use fuzzy matching on alert summary/title/description:

   **Matching rules:**
   - Case-insensitive keyword matching
   - Match any keyword from the list to identify the runbook
   - If multiple runbooks match, prefer the one with more keyword hits
   - If no match found, list all available runbooks for manual selection

   See **Known Runbooks** section below for keyword mappings.

3. **Fetch the runbook content from GitHub:**
   ```bash
   gh api repos/${RUNBOOK_REPO}/contents/<filename>?ref=${RUNBOOK_BRANCH} --jq '.content' | base64 -d
   ```

4. **Return the full runbook content** including:
   - When the alert fires
   - Owning team and Slack channel
   - All steps to follow
   - Any warnings or critical notes

## Output Format

```
## Runbook: <title>

**Source:** https://github.com/${RUNBOOK_REPO}/blob/${RUNBOOK_BRANCH}/<filename>
**Owning team:** <team> (<slack-channel>)

### When it fires
<condition>

### Steps
<numbered steps from runbook>

### Warnings
<any warnings or critical notes>
```

## Known Runbooks

| Runbook | Keywords | Owning Team |
|---------|----------|-------------|
| `disk-pressure.md` | disk, storage, full, capacity, space, 90%, 95%, pressure | Data (`#orbit-data`) |
| `heartbeat-stale.md` | heartbeat, stale, worker, offline, unresponsive, timeout | Workers (`#orbit-workers`) |
| `gateway-health.md` | gateway, health, check, failed, http, endpoint, 5xx, 4xx | Platform (`#orbit-platform`) |
| `price-feed-missing.md` | price, feed, missing, oracle, stale, outdated, market | Markets (`#orbit-markets`) |
| `rpc-lag.md` | rpc, lag, latency, slow, delay, response, node | Platform (`#orbit-platform`) |
| `validator-jailed.md` | validator, jailed, slashed, offline, missed, blocks | Validators (`#orbit-validators`) |

### Adding New Runbook Mapping

To add a new runbook mapping:

1. **Add the runbook file** to the repository:
   ```
   ${RUNBOOK_REPO}/<new-runbook>.md
   ```

2. **Update this table** with:
   - Runbook filename
   - Keywords (lowercase, comma-separated) - include alert names, error messages, symptoms
   - Owning team with Slack channel

3. **Keyword selection tips:**
   - Include the alert name/title words
   - Add common error messages or symptoms
   - Include metric names if applicable
   - Add abbreviations (e.g., "rpc" for "Remote Procedure Call")

## Error Handling

### GitHub API unavailable
If `gh api` fails with connection error:
1. Return error: `ERROR: GitHub API unavailable`
2. Provide manual lookup instructions:
   ```
   Manual lookup required:
   1. Open https://github.com/${RUNBOOK_REPO}
   2. Switch to branch: ${RUNBOOK_BRANCH}
   3. Find the runbook: <filename>
   4. Copy the relevant sections for triage
   ```

### Runbook not found (404)
If the runbook file doesn't exist:
1. Return error: `ERROR: Runbook not found - <filename>`
2. List available runbooks from the repository
3. Suggest checking if the alert has incorrect runbook reference

### Rate limit exceeded (403)
If GitHub API returns rate limit error:
1. Wait 30 seconds and retry (up to 2 attempts)
2. If still failing after retries, try alternative method using raw URL:
   ```bash
   curl -s "https://raw.githubusercontent.com/${RUNBOOK_REPO}/${RUNBOOK_BRANCH}/<filename>"
   ```
3. If all methods fail, return error:
   ```
   ERROR: GitHub rate limit exceeded
   Attempted: 2 retries with 30s delay
   Fallback options:
   - Use raw URL: https://raw.githubusercontent.com/${RUNBOOK_REPO}/${RUNBOOK_BRANCH}/<filename>
   - Manual lookup at: https://github.com/${RUNBOOK_REPO}
   ```

## Tools Used

| Tool | Purpose |
|------|---------|
| `Bash` | Execute `gh api` and `curl` commands to fetch runbooks from GitHub |
| `Read` | Read local alert JSON to extract `alert_fields.runbook` URL |

## Security Notes

- This skill is READ-ONLY
- It ONLY fetches runbook content from GitHub
- It does NOT execute any scripts or commands mentioned in runbooks
