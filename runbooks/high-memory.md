# Runbook: High Memory Usage

**Alert:** `HighMemoryUsage`
**Severity:** Warning
**Threshold:** Memory > 90% for 5 minutes

---

## What It Means

Available memory on a host is critically low. This can cause the OS to start using swap space, significantly degrading performance. If memory is exhausted completely, the OOM (Out of Memory) killer will start terminating processes.

---

## Immediate Actions (first 5 minutes)

1. Acknowledge the alert in PagerDuty / Opsgenie
2. Check Grafana — is memory still rising or has it stabilised?
3. SSH into the affected host:

```bash
ssh <hostname>
```

4. Check current memory state:

```bash
free -h
```

5. Check if OOM killer has already triggered:

```bash
dmesg | grep -i "oom\|killed process" | tail -20
journalctl -k | grep -i "oom" | tail -20
```

---

## Investigation Steps

### Identify top memory consumers

```bash
# Processes sorted by memory
ps aux --sort=-%mem | head -10

# Detailed memory map per process
cat /proc/<PID>/status | grep -i vmrss

# Check for memory leaks over time
watch -n 5 'ps aux --sort=-%mem | head -10'
```

### Check swap usage

```bash
# Swap usage overview
swapon --show
vmstat -s | grep swap

# Processes using swap
for pid in /proc/[0-9]*; do
  comm=$(cat $pid/comm 2>/dev/null)
  swap=$(grep VmSwap $pid/status 2>/dev/null | awk '{print $2}')
  [ "${swap:-0}" -gt 0 ] && echo "$comm: ${swap}kB"
done | sort -t: -k2 -rn | head -10
```

### Check memory trending

```bash
# Memory usage over last hour in Grafana
# Dashboard: Node Exporter Full → Memory section

# Or via CLI
sar -r 1 10
```

---

## Resolution

### If caused by a memory leak in a service

```bash
# Restart the offending service
systemctl restart <service-name>

# Verify memory drops after restart
watch -n 2 free -h
```

### If swap is exhausted

```bash
# Clear swap (only if memory allows)
swapoff -a && swapon -a
```

### If OOM killer has triggered

```bash
# Check which process was killed
dmesg | grep "Killed process"

# Restart the killed service if needed
systemctl restart <service-name>
```

---

## Escalation

| Condition | Escalate To |
|-----------|-------------|
| Memory > 95% and rising | Senior Engineer on-call |
| OOM killer triggering repeatedly | Engineering Lead |
| Production service killed by OOM | Immediate escalation to Team Lead |

---

## Post-Incident

- Document which process consumed memory and why
- Consider increasing instance memory if recurring
- Add memory leak detection to code review checklist if software related
- Update this runbook if new steps were discovered
