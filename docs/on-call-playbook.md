# On-Call Playbook

A practical guide for engineers on the on-call rotation — covering responsibilities, tools, escalation, and best practices.

---

## On-Call Responsibilities

- Monitor alerts via PagerDuty / Opsgenie during your shift
- Acknowledge alerts within the defined SLA
- Investigate and resolve incidents or escalate appropriately
- Document all actions taken during an incident
- Hand off open incidents at shift change with full context

---

## Before Your Shift

```
✅ Confirm you have access to all required systems
✅ Verify PagerDuty / Opsgenie is configured on your phone
✅ Review any open incidents or known issues from previous shift
✅ Check recent deployments that could cause instability
✅ Ensure VPN access is working
✅ Know who to escalate to
```

---

## When an Alert Fires

### Step 1 — Acknowledge (within SLA)
- Acknowledge in PagerDuty / Opsgenie immediately
- Do not let alerts auto-escalate unnecessarily

### Step 2 — Assess severity
- Read the alert name and description
- Check Grafana for context and correlated alerts
- Determine P1 / P2 / P3 / P4 severity

### Step 3 — Investigate
- Follow the relevant runbook
- Document every action in the incident ticket
- Keep a running timeline

### Step 4 — Communicate
- P1/P2 → Post in Slack incident channel immediately
- Update status page if users are affected
- Keep stakeholders informed every 30 minutes on P1

### Step 5 — Resolve or Escalate
- Apply the fix from the runbook
- If stuck after 30 minutes → escalate
- Never sit on a P1 alone

### Step 6 — Post-incident
- Close the incident ticket with resolution notes
- Create RCA for P1/P2 incidents
- Create Jira tickets for follow-up action items

---

## Key Commands Reference

```bash
# Check system health
top / htop
free -h
df -h
uptime

# Check service status
systemctl status <service>
journalctl -u <service> -n 100

# Check network
ip addr show
ss -tlnp
ping <host>
traceroute <host>

# Check GCP instance
gcloud compute instances list
gcloud compute instances describe <name> --zone=<zone>
gcloud compute instances start <name> --zone=<zone>

# Docker
docker ps
docker logs <container>
docker restart <container>
```

---

## Escalation Contacts

| Role | When to Escalate |
|------|-----------------|
| Senior Engineer | P1 unresolved after 15 min, unknown root cause |
| Engineering Lead | P1 with business impact, multiple services down |
| Database Admin | Any database-related P1/P2 |
| Network Engineer | Network-wide connectivity issues |
| Security Team | Any suspected security incident |

---

## Shift Handoff Checklist

```
✅ Document all open incidents with current status
✅ Note any systems in degraded state
✅ List any temporary fixes that need permanent resolution
✅ Share any useful context discovered during shift
✅ Confirm incoming engineer has acknowledged handoff
```

---

## Golden Rules

1. **Acknowledge fast** — silence causes unnecessary escalation
2. **Communicate early** — stakeholders hate surprises
3. **Follow the runbook** — don't improvise under pressure
4. **Document everything** — your notes help the RCA
5. **Escalate without hesitation** — it's not a sign of weakness
6. **Blameless mindset** — focus on fixing systems, not blaming people
