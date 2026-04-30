# incident-runbooks

A structured collection of incident response runbooks, RCA templates, and on-call playbooks — built for NOC and infrastructure engineering teams.

## Structure

```
incident-runbooks/
├── runbooks/
│   ├── high-cpu.md
│   ├── high-memory.md
│   ├── disk-space-critical.md
│   ├── service-down.md
│   └── network-connectivity.md
├── templates/
│   ├── rca-template.md
│   └── incident-report-template.md
└── docs/
    ├── incident-severity-levels.md
    └── on-call-playbook.md
```

## Purpose

These runbooks are designed to enable fast, consistent incident response across teams and shifts. Each runbook follows the same structure:

- **What it means** — plain explanation of the alert
- **Immediate actions** — what to do in the first 5 minutes
- **Investigation steps** — how to diagnose root cause
- **Resolution** — how to fix it
- **Escalation** — when and who to escalate to

## Principles

- Speed over perfection in the first 5 minutes
- Document everything during the incident
- Blameless post-mortems — focus on systems, not people
- Every incident is an opportunity to improve
