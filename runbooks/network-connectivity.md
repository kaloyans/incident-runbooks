# Runbook: Network Connectivity Issues

**Alert:** `NetworkConnectivityDegraded` / `HighPacketLoss`
**Severity:** Warning / Critical
**Threshold:** Packet loss > 5% or latency > 200ms sustained

---

## What It Means

A host or service is experiencing network connectivity problems. This can manifest as slow response times, intermittent connectivity, or complete network isolation. Root causes range from misconfigured routes to hardware failures.

---

## Immediate Actions (first 5 minutes)

1. Acknowledge the alert in PagerDuty / Opsgenie
2. Identify affected host and scope — is it one host or multiple?
3. Check Grafana — network error rate and latency graphs
4. Test basic connectivity from a healthy host:

```bash
ping -c 10 <affected-host>
```

5. Determine scope:

```bash
# Test gateway
ping -c 4 <gateway-ip>

# Test external connectivity
ping -c 4 8.8.8.8

# Test DNS resolution
dig google.com
```

---

## Investigation Steps

### Check interface status

```bash
# List all interfaces and their state
ip link show

# Check interface statistics — errors and drops
ip -s link show <interface>

# Check IP addresses
ip addr show
```

### Check routing

```bash
# View routing table
ip route show

# Trace route to destination
traceroute <destination>
mtr --report <destination>
```

### Check for packet loss

```bash
# Extended ping test
ping -c 100 <destination>

# Monitor in real time
mtr <destination>
```

### Check firewall rules

```bash
# List iptables rules
iptables -L -n -v

# Check if traffic is being dropped
journalctl | grep iptables-dropped | tail -20

# Temporarily disable firewall for testing (use with caution)
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
```

### Check DNS

```bash
# Test DNS resolution
dig @8.8.8.8 google.com
dig @192.168.1.10 example.com

# Check /etc/resolv.conf
cat /etc/resolv.conf

# Check systemd-resolved
systemd-resolve --status
```

### Capture traffic for analysis

```bash
# Capture traffic on interface
tcpdump -i eth0 -n -c 100

# Capture traffic to/from specific host
tcpdump -i eth0 host <destination> -n

# Save capture for analysis
tcpdump -i eth0 -w /tmp/capture.pcap
```

---

## Resolution

### If routing is misconfigured

```bash
# Add missing route
ip route add <destination>/<prefix> via <gateway>

# Make persistent
# Edit /etc/network/interfaces or /etc/netplan/*.yaml
```

### If DNS is broken

```bash
# Restart systemd-resolved
systemctl restart systemd-resolved

# Flush DNS cache
systemd-resolve --flush-caches

# Temporarily use public DNS
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

### If firewall is blocking traffic

```bash
# Allow specific port
iptables -A INPUT -p tcp --dport <port> -j ACCEPT

# Save rules
netfilter-persistent save
```

### If network interface is down

```bash
# Bring interface up
ip link set <interface> up

# Restart networking
systemctl restart networking
```

---

## Escalation

| Condition | Escalate To |
|-----------|-------------|
| Multiple hosts affected | Network Engineer immediately |
| Gateway unreachable | Infrastructure Lead |
| ISP or upstream connectivity lost | Network Engineer + Management |
| VPN tunnel down affecting remote access | Network Engineer on-call |

---

## Post-Incident

- Document which layer the issue was at (physical, network, transport)
- Review firewall rule changes made recently
- Check if BGP routes or OSPF topology changed
- Update network diagrams if topology changed
- Consider adding synthetic monitoring for critical paths
