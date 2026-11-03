# Alert hb7Qx2: HeartbeatStale on worker-2

## What is happening

Worker-2 stopped sending heartbeats due to persistent queue publish timeouts. The worker receives events (request_rate 0.5 req/s) but cannot publish responses or heartbeats (response_rate 0 req/s). This is the third worker affected by this pattern in the last 24 hours, indicating a systemic issue with the message queue.

## Evidence

| File | Finding |
|------|---------|
| `evidence/hb7Qx2/metrics.txt` | worker-2 last heartbeat 19.2h ago, request_rate 0.5 req/s, response_rate **0 req/s**, 11 reconnects in 15 min. Other workers healthy. |
| `evidence/hb7Qx2/worker-2.log` | Last successful heartbeat at 07:44:12 (seq=88413). After that: continuous "queue publish timeout after 20s" errors, multiple reconnect cycles. Still timing out at 08:09:45. |
| `evidence/hb7Qx2/handover-notes.md` | **Previous shift (Nov 1, 16:37 UTC)**: Same pattern on worker-1 and worker-3. Restarted both, cleared by 17:05. Posted to #orbit-workers - **root cause not yet found**. |

## Decision

**ESCALATE**

Per runbook Step 4: "Escalate to `#orbit-workers` if more than one worker was affected in the last 24 hours. This points to a shared cause (the message queue), and restarts only hide it."

Evidence shows:
- Nov 1, 16:37: worker-1 and worker-3 affected (same pattern)
- Nov 2, 07:45: worker-2 affected (same pattern)
- Total: 3 workers in <24 hours = systemic issue

## Action / Escalation Note

Post to `#orbit-workers`:
```
HeartbeatStale on worker-2 (mainnet) - ESCALATING

This is the 3rd worker affected in <24h (worker-1, worker-3 yesterday per handover notes).

Pattern: queue publish timeouts, worker receives events but cannot publish.
- Last heartbeat: 07:44:12 UTC
- Logs show continuous "queue publish timeout after 20s" since 07:45:10
- 11 reconnects in 15 min, response_rate 0

Previous shift restarted worker-1/worker-3 yesterday but no root cause found.
Repeated occurrence points to message queue issue.

Logs attached: worker-2.log, metrics snapshot
```

## Runbook Reference

**Runbook:** [HeartbeatStale](https://github.com/techops-services/orbit-runbooks/blob/main/heartbeat-stale.md)
**Section:** Step 4 - Escalate (more than one worker in 24h)
**Owning team:** Workers (`#orbit-workers`)

## Unknowns

- Root cause of queue publish timeouts not yet identified (team is investigating per handover notes)
- Why worker-2 specifically hit this now while others recovered?
- Message queue health/metrics not available in evidence
