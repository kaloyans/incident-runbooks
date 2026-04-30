# Runbook: High CPU Usage

**Alert:** `HighCpuUsage`
**Severity:** Warning
**Threshold:** CPU > 85% for 5 minutes

---

## What It Means

One or more servers are experiencing sustained high CPU utilisation. This can lead to slow response times, request timeouts, and eventual service degradation if not addressed.

---

## Immediate Actions (first 5 minutes)

1. Acknowledge the alert in PagerDuty / Opsgenie
2. Identify which host is affected from the alert details
3. Check Grafana dashboard for CPU trend — is it still rising or stabilising?
4. SSH into the affected host:

```bash
ssh <hostname>
```

5. Get a quick overview:

```bash
top -b -n 1 | head -20
```

---

## Investigation Steps

### Identify the offending process

```bash
# Top processes by CPU
ps aux --sort=-%cpu | head -10

# Real-time process monitor
htop

# Check if a specific service is spiking
systemctl status <service-name>
```

### Check for recent changes

```bash
# Recent deployments or cron jobs
journalctl --since "30 minutes ago" | grep -i "deploy\|cron\|start\|restart"

# Check cron jobs
crontab -l
cat /etc/crontab
```

### Check system load history

```bash
# Load average history
uptime
sar -u 1 10

# Check if load is from user or system space
vmstat 1 5
```

---

## Resolution

### If caused by a runaway process

```bash
# Graceful restart of the service
systemctl restart <service-name>

# If unresponsive — force kill
kill -9 <PID>
```

### If caused by legitimate load spike

- Notify the development team
- Consider scaling the instance vertically or horizontally
- Monitor for 15 minutes after the spike

### If caused by a cron job

```bash
# Identify and reschedule the job to off-peak hours
crontab -e
```

---

## Escalation

| Condition | Escalate To |
|-----------|-------------|
| CPU > 95% for more than 15 minutes | Senior Engineer on-call |
| Production service impacted | Engineering Lead |
| Unknown root cause after 30 minutes | Team Lead |

---

## Post-Incident

- Document findings in the incident ticket
- If recurring — open a Problem ticket for root cause investigation
- Update this runbook if new steps were discovered
