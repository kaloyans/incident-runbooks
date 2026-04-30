# Incident Severity Levels

A clear and consistent severity framework ensures the right people are notified at the right time and that response effort matches business impact.

---

## Severity Matrix

| Level | Name | Description | Examples |
|-------|------|-------------|---------|
| **P1** | Critical | Complete service outage or data loss. All users affected. Revenue impact. | Production database down, complete API failure, security breach |
| **P2** | High | Major feature unavailable or severe degradation. Significant user impact. | Payment processing failing, login broken for 50%+ users |
| **P3** | Medium | Partial degradation. Some users or features affected. Workaround available. | Slow API responses, non-critical feature down, single region affected |
| **P4** | Low | Minor issue. Minimal user impact. No immediate action required. | UI glitch, non-critical alert firing, cosmetic bug |

---

## Response Times

| Severity | Acknowledge | Start Investigation | Resolve |
|----------|-------------|--------------------|---------| 
| P1 | 5 minutes | 15 minutes | Best effort — all hands |
| P2 | 15 minutes | 30 minutes | 4 hours |
| P3 | 1 hour | 4 hours | 24 hours |
| P4 | Next business day | Next business day | 1 week |

---

## Escalation Path

### P1 — Critical
```
Alert fires
    → On-call Engineer (immediate)
        → Engineering Lead (if not resolved in 15 min)
            → CTO / Management (if not resolved in 30 min)
```

### P2 — High
```
Alert fires
    → On-call Engineer (15 min)
        → Senior Engineer (if not resolved in 1 hour)
```

### P3 — Medium
```
Alert fires
    → On-call Engineer (1 hour)
        → Team Lead (if not resolved in 4 hours)
```

### P4 — Low
```
Ticket created
    → Assigned Engineer
        → Next sprint if not urgent
```

---

## Severity Upgrade Criteria

Upgrade severity if any of the following occur:

- Impact is wider than initially assessed
- Resolution time is exceeding SLA
- Data integrity is at risk
- Security implications discovered
- Customer complaints increasing rapidly

---

## Communication Requirements

| Severity | Internal Slack | Status Page | Customer Email | Management |
|----------|---------------|-------------|----------------|------------|
| P1 | Immediate | Yes | Yes (if > 30 min) | Yes |
| P2 | Immediate | Yes | Only if requested | On resolution |
| P3 | Within 1 hour | Optional | No | No |
| P4 | Ticket only | No | No | No |
