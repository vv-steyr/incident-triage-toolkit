# Alert or8Tz5: Price feed missing for SOL-PERP (testnet)

## What is happening

Price feed for SOL-PERP market on testnet is stale - no valid price updates for 25 minutes. The cause is "vendor stale" - the external price vendor stopped sending updates. Trading on SOL-PERP is automatically paused. Other markets (BTC-PERP, ETH-PERP) are unaffected.

## Evidence

| File | Finding |
|------|---------|
| `evidence/or8Tz5/oracle-status.txt` | SOL-PERP: 2 sources configured, **0 valid**, last vendor update **25 min ago**, status "NO PRICE (vendor stale)". BTC-PERP and ETH-PERP both OK with recent updates (6-8s ago). |

## Decision

**MONITOR**

Per runbook:
- Step 1: This is **testnet** → "Do not page anyone"
- Step 2: Cause is "vendor stale" → "usually recovers within 30 minutes"

The vendor has been stale for 25 minutes. Per runbook guidance, it should recover within ~30 minutes. Post notification to testnet channel and monitor.

## Action / Escalation Note

Post to `#orbit-markets-testnet`:
```
Price feed missing: SOL-PERP (testnet)

Cause: vendor stale (last update 25 min ago)
Impact: SOL-PERP trading paused, other markets unaffected
Status: Monitoring - vendor usually recovers within 30 min

Will update if not resolved by 08:35 UTC.
```

If not resolved by 08:35 UTC (30 min threshold), escalate to `#orbit-markets` for vendor investigation.

## Runbook Reference

**Runbook:** [Price feed missing](https://github.com/techops-services/orbit-runbooks/blob/main/price-feed-missing.md)
**Section:** Step 1 (Testnet) + Step 2 (vendor stale)
**Owning team:** Markets (`#orbit-markets`)

## Unknowns

- Which specific vendor(s) are stale? (only 2 sources configured for SOL-PERP vs 3 for BTC/ETH)
- Why does SOL-PERP have fewer price sources than other markets?
- Vendor status page or external monitoring not available in evidence
