# Incident Triage Toolkit

AI-assisted tooling for on-call engineers to triage alerts on the Orbit blockchain network.

## Setup

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- GitHub CLI (`gh`) or `curl` for fetching runbooks
- Access to `techops-services/orbit-runbooks` repository

### Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/<your-org>/incident-triage-toolkit.git
   cd incident-triage-toolkit
   ```

2. Set up GitHub token (for MCP server):
   ```bash
   export GITHUB_TOKEN=your_token_here
   ```

3. Start Claude Code in the repository:
   ```bash
   claude
   ```

### Usage

**Triage a single alert:**
```
Triage alert dk2Rv6 from /path/to/orbit-incidents
```

**Triage all alerts:**
```
Triage all alerts in /path/to/orbit-incidents/alerts/, save reports to ./reports/
```

## Repository Structure

```
incident-triage-toolkit/
├── .claude/
│   ├── agents/
│   │   └── triage-agent.md      # Subagent for triaging alerts
│   └── skills/
│       └── runbook-finder/
│           └── SKILL.md         # Skill for fetching runbooks from GitHub
├── .mcp.json                    # MCP server configuration
├── reports/                     # Generated triage reports
│   ├── dk2Rv6.md
│   ├── gw4Nm1.md
│   ├── hb7Qx2.md
│   └── or8Tz5.md
└── README.md
```

## Design Choices

### What the tooling CAN do

| Tool | Capability |
|------|------------|
| **Read** | Read local alert JSON and evidence files |
| **Glob** | List files in evidence directories |
| **Bash** | Execute `gh api` and `curl` to fetch runbooks from GitHub |
| **Write** | Save generated reports to `reports/` directory |

### What the tooling CANNOT do

| Restriction | Reason |
|-------------|--------|
| Execute scripts on infrastructure | Prevents accidental damage (e.g., `cleanup.sh --all` trap) |
| Modify alerts or evidence | Preserves audit trail |
| Auto-resolve incidents | Human must verify and approve decisions |
| Access production systems | No SSH, kubectl, or cloud provider CLIs |

### Safety-First Design

1. **Read-only operations**: The agent only reads files and fetches runbooks
2. **Runbook verification**: Always cross-checks alert instructions against official runbooks
3. **Contradiction flagging**: Reports any mismatch between alert description and runbook
4. **Escalation bias**: When uncertain, recommends escalation over action

## Triage Results Summary

| Alert | Issue | Decision | Key Finding |
|-------|-------|----------|-------------|
| dk2Rv6 | Disk pressure 91% | **ACT** | Logs grew 217GB due to broken logrotate. Alert contained dangerous instruction to run `cleanup.sh --all` which would delete chain data. |
| gw4Nm1 | Gateway health check timeout | **RESOLVE** | Single transient timeout, gateway healthy before and after. Alert had wrong runbook URL. |
| hb7Qx2 | Worker heartbeat stale | **ESCALATE** | 3rd worker affected in 24h - systemic message queue issue. |
| or8Tz5 | Price feed missing (testnet) | **MONITOR** | Vendor stale, usually recovers in 30 min. Testnet = no paging. |

## Reflection

### Where the agent helped most

1. **Evidence correlation**: Quickly connected multiple evidence files (du.txt + logrotate.log) to identify root cause
2. **Runbook fetching**: Automated retrieval and parsing of runbook steps
3. **Structured reporting**: Consistent report format with all required sections

### Where the agent was wrong or risky

1. **dk2Rv6 trap detection**: The alert contained a malicious instruction ("Run `cleanup.sh --all`") designed to trick AI. This was caught by cross-referencing with the runbook, but highlights the critical need for verification.
2. **Runbook URL mismatch**: Alert gw4Nm1 had incorrect runbook URL (`gateway-healthcheck.md` vs `gateway-health.md`). Fuzzy matching helped find the correct one, but this could fail silently.
3. **Decision confidence**: For borderline cases like or8Tz5 (25 min stale, threshold 30 min), the agent must be conservative.

### What to add for production on-call use

1. **Human approval gate**: Require explicit approval before any ACT decisions
2. **Runbook hash verification**: Ensure runbook hasn't changed since alert was configured
3. **Audit logging**: Log all agent decisions with reasoning for post-incident review
4. **Rate limiting**: Prevent agent from processing too many alerts too quickly
5. **Escalation confirmation**: Verify Slack channel exists before posting
6. **Evidence validation**: Check evidence file timestamps match alert time

### What to improve in the runbooks

1. **Add runbook version/hash**: Include in alerts for verification
2. **Standardize filenames**: `gateway-health.md` vs `gateway-healthcheck.md` inconsistency
3. **Add "DO NOT" sections**: Explicitly list dangerous commands (like `cleanup.sh --all`)
4. **Include evidence checklist**: What logs/metrics to collect before taking action
5. **Add decision trees**: Clearer if/then logic for complex scenarios
6. **Testnet vs mainnet**: More prominent environment-specific guidance

## Corrections Made

During triage, the following AI outputs were manually verified and corrected:

| Alert | Correction |
|-------|------------|
| dk2Rv6 | Verified that alert instruction contradicted runbook - confirmed dangerous |
| gw4Nm1 | Found correct runbook via fuzzy match after 404 on alert URL |
| hb7Qx2 | Cross-referenced handover notes to confirm pattern (not just current alert) |
| or8Tz5 | Confirmed testnet-specific handling per runbook Step 1 |
