# Runbook: Disk Space Critical

**Alert:** `DiskSpaceCritical`
**Severity:** Critical
**Threshold:** Disk usage > 85% on any filesystem

---

## What It Means

A filesystem is running out of space. If disk reaches 100%, services will fail to write logs, databases will crash, and the OS may become unstable. This requires immediate action.

---

## Immediate Actions (first 5 minutes)

1. Acknowledge the alert in PagerDuty / Opsgenie
2. Identify which host and filesystem is affected from the alert
3. SSH into the affected host:

```bash
ssh <hostname>
```

4. Check disk usage across all filesystems:

```bash
df -h
```

5. If any filesystem is above 95% — treat as **Critical** and act immediately.

---

## Investigation Steps

### Find what is consuming disk space

```bash
# Top directories by size from root
du -sh /* 2>/dev/null | sort -rh | head -15

# Drill down into the largest directory
du -sh /var/* 2>/dev/null | sort -rh | head -10

# Find largest individual files
find / -type f -size +500M 2>/dev/null | xargs du -sh | sort -rh | head -10
```

### Check logs

```bash
# Log directory size
du -sh /var/log/*  | sort -rh | head -10

# Check for abnormally large log files
find /var/log -type f -size +100M 2>/dev/null
```

### Check for deleted files still held open

```bash
# Files deleted but still held by a process (consuming space)
lsof | grep deleted | sort -k7 -rn | head -10
```

---

## Resolution

### Clean up logs

```bash
# Rotate logs immediately
logrotate -f /etc/logrotate.conf

# Vacuum systemd journal
journalctl --vacuum-size=500M
journalctl --vacuum-time=7d

# Remove old compressed logs
find /var/log -name "*.gz" -mtime +7 -delete
```

### Clean up Docker (if applicable)

```bash
# Remove unused images, containers, volumes
docker system prune -a --volumes
```

### Clean up package cache

```bash
# Debian/Ubuntu
apt-get clean
apt-get autoremove -y

# RHEL/CentOS
yum clean all
dnf clean all
```

### Clean up temp files

```bash
rm -rf /tmp/*
rm -rf /var/tmp/*
```

### If a specific application is filling disk

```bash
# Truncate a large log file without deleting it
truncate -s 0 /var/log/<application>.log

# Restart the application to reset logging
systemctl restart <service-name>
```

---

## Escalation

| Condition | Escalate To |
|-----------|-------------|
| Disk > 95% and cannot free space quickly | Senior Engineer on-call |
| Database filesystem full | Database Administrator immediately |
| Root filesystem full | Immediate escalation — host may be unstable |

---

## Post-Incident

- Identify root cause — abnormal log growth, missing rotation, runaway process
- Implement log rotation if not already configured
- Set up disk space trending alerts at 70% and 80% as early warning
- Consider expanding disk or archiving old data if recurring
