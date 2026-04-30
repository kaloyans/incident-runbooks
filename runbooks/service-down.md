# Runbook: Service Down

**Alert:** `InstanceDown` / `ServiceUnreachable`
**Severity:** Critical
**Threshold:** Service unreachable for more than 1 minute

---

## What It Means

A monitored service or host has stopped responding. This could be caused by a crashed process, failed deployment, network issue, or host-level problem.

---

## Immediate Actions (first 5 minutes)

1. Acknowledge the alert in PagerDuty / Opsgenie
2. Identify which service and host is affected
3. Check Grafana — when did the service go down? Any correlated alerts?
4. Attempt to reach the host:

```bash
ping -c 4 <hostname>
```

5. Attempt SSH:

```bash
ssh <hostname>
```

---

## Investigation Steps

### If host is reachable via SSH

```bash
# Check service status
systemctl status <service-name>

# Check recent logs
journalctl -u <service-name> -n 100 --no-pager

# Check if process is running
ps aux | grep <service-name>

# Check if port is listening
ss -tlnp | grep <port>

# Check system resources
free -h
df -h
uptime
```

### If host is not reachable

```bash
# Try from another host in the same network
ping -c 4 <hostname>
traceroute <hostname>

# Check GCP console for instance status
gcloud compute instances describe <instance-name> --zone=<zone>

# Check if instance was stopped or preempted
gcloud compute instances list | grep <instance-name>
```

### Check for recent changes

```bash
# Recent deployments
journalctl --since "1 hour ago" | grep -i "deploy\|restart\|stop\|start"

# Recent system changes
last reboot
```

---

## Resolution

### Restart the failed service

```bash
systemctl restart <service-name>
systemctl status <service-name>

# Verify port is now listening
ss -tlnp | grep <port>

# Test service response
curl -I http://localhost:<port>/health
```

### If host is stopped in GCP

```bash
# Start the instance
gcloud compute instances start <instance-name> --zone=<zone>

# Monitor startup
gcloud compute instances get-serial-port-output <instance-name> --zone=<zone>
```

### If caused by a failed deployment

```bash
# Roll back to previous version
systemctl stop <service-name>
# Restore previous binary or container image
systemctl start <service-name>
```

---

## Escalation

| Condition | Escalate To |
|-----------|-------------|
| Service cannot be restarted | Senior Engineer on-call |
| Host unreachable and not recoverable via GCP console | Infrastructure Lead |
| Multiple services down simultaneously | Incident Commander — declare Major Incident |
| Production database down | Database Administrator immediately |

---

## Post-Incident

- Document exact timeline — when did it go down, when detected, when resolved
- Identify root cause — crash, OOM, deployment failure, infrastructure issue
- Review monitoring coverage — was detection fast enough?
- Update deployment process if deployment caused the outage
