# Alert gw4Nm1: Gateway health check failed

## What is happening

A single health check timeout (10s) occurred at 07:58:00 UTC on api-gateway. All checks before and after this event passed successfully with normal latency (~140ms). This is a transient network issue.

## Evidence

| File | Finding |
|------|---------|
| `evidence/gw4Nm1/healthchecks.csv` | Only 1 failure at 07:58:00 (timeout). All other checks pass: 07:55-07:57 pass, 07:59-08:15 all pass with ~140ms latency. |
| `evidence/gw4Nm1/manual-check.txt` | Manual check at 08:16:05 UTC returned HTTP 200. Gateway is healthy. |

## Decision

**RESOLVE**

Per runbook Step 2: "One failure, then passes: treat as transient. Resolve the alert with a note. No escalation."

The evidence shows exactly this pattern - a single timeout surrounded by successful checks on both sides, and a manual verification confirming the gateway is healthy.

## Action / Escalation Note

Resolve with note:
```
Transient health check timeout at 07:58 UTC. Single failure among continuous passes before and after. Manual verification at 08:16 confirmed gateway healthy (HTTP 200). No action required.
```

## Runbook Reference

**Runbook:** [Gateway health check failed](https://github.com/techops-services/orbit-runbooks/blob/main/gateway-health.md)
**Section:** Step 2 - "One failure, then passes"
**Owning team:** Platform (`#orbit-platform`)

## Unknowns

- **Incorrect runbook URL in alert**: Alert references `gateway-healthcheck.md` which returns 404. Correct runbook is `gateway-health.md`. The alert configuration should be updated.
- Root cause of the single timeout unknown (network blip, momentary load spike, etc.) but not actionable given transient nature.
