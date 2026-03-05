Failover-Lab 
# Failover Lab — Active-Passive Failover with Keepalived and VRRP

A hands-on lab where I built an active-passive failover system using Keepalived on two Linux virtual machines. A Virtual IP (VIP) floats between a primary and backup server. When the primary goes down, the backup automatically picks up the VIP and traffic continues uninterrupted.

---

## Lab Environment

| Role | OS | IP | VIP |
|---|---|---|---|
| Primary Server | Ubuntu 24.04 | 192.168.56.101 | 192.168.56.200 |
| Backup Server | Kali Linux | 192.168.56.103 | 192.168.56.200 |

**Virtualization:** VirtualBox with Host-Only Adapter for VM-to-VM communication

---

## What I Built

- An **active-passive failover** system using Keepalived and VRRP protocol
- A **Virtual IP (VIP)** that floats between primary and backup server
- **Automatic failover** — VIP moves to backup within seconds when primary goes down
- **Automatic failback** — VIP returns to primary when it recovers
- **Full failover cycle test** — simulated failure, confirmed traffic continuity, confirmed recovery

---

## Network Diagram

```
                        Virtual IP: 192.168.56.200
                                    |
              ----------------------|----------------------
              |                                           |
+-------------------+                       +-------------------+
|   Ubuntu 24.04    |   VRRP Heartbeat      |    Kali Linux     |
|   PRIMARY         | <-------------------> |    BACKUP         |
|  192.168.56.101   |   every 1 second      |  192.168.56.103   |
|   priority 100    |                       |   priority 50     |
+-------------------+                       +-------------------+

Normal state:   VIP on Ubuntu  (priority 100 wins)
Failure state:  VIP on Kali    (Ubuntu heartbeat stops, Kali takes over)
Recovery state: VIP on Ubuntu  (Ubuntu comes back, reclaims VIP)
```

---

## How VRRP Works

VRRP (Virtual Router Redundancy Protocol) is the protocol Keepalived uses to manage failover:

- Both servers join the same VRRP instance with a shared `virtual_router_id`
- The server with the **highest priority** holds the Virtual IP
- Every second, the primary sends a **heartbeat** to announce it's alive
- If the backup stops receiving heartbeats, it assumes the primary is down and **takes over the VIP**
- When the primary comes back, it reclaims the VIP because its priority is higher

---

## Steps Taken

### 1. Installed Keepalived on Both Machines
```bash
# On Ubuntu and Kali
sudo apt install keepalived -y
```

### 2. Configured Primary Server (Ubuntu)
Created `/etc/keepalived/keepalived.conf` — see [configs/primary-keepalived.conf](configs/primary-keepalived.conf)

### 3. Configured Backup Server (Kali)
Created `/etc/keepalived/keepalived.conf` — see [configs/backup-keepalived.conf](configs/backup-keepalived.conf)

### 4. Started Keepalived on Both Machines
```bash
sudo systemctl start keepalived
sudo systemctl status keepalived
```

### 5. Verified VIP on Primary
```bash
ip a show enp0s8
# Result: 192.168.56.200 listed as secondary IP on Ubuntu
```

### 6. Tested Connectivity to VIP
```bash
ping 192.168.56.200
# Result: successful replies from Ubuntu
```

### 7. Simulated Primary Failure
```bash
# On Ubuntu - simulate server going down
sudo systemctl stop keepalived

# On Kali - verify VIP moved over
ip a show eth1
# Result: 192.168.56.200 now on Kali
```

### 8. Verified Traffic Continuity During Failover
```bash
ping 192.168.56.200
# Result: still pinging - Kali now responding to VIP
```

### 9. Tested Failback (Primary Recovery)
```bash
# On Ubuntu - bring primary back
sudo systemctl start keepalived

# Verify VIP returned to Ubuntu
ip a show enp0s8
# Result: 192.168.56.200 back on Ubuntu
```

---

## Full Failover Cycle Results

| Event | VIP Location | Traffic |
|---|---|---|
| Both servers running | Ubuntu (priority 100) | ✅ Pinging |
| Ubuntu keepalived stopped | Kali (took over automatically) | ✅ Still pinging |
| Ubuntu keepalived restarted | Ubuntu (reclaimed automatically) | ✅ Still pinging |

---

## Key Concepts Learned

- **Keepalived** — Linux tool for managing high availability using VRRP
- **VRRP (Virtual Router Redundancy Protocol)** — protocol that manages which server holds the VIP
- **Virtual IP (VIP)** — a floating IP address not tied to any single machine
- **Active-Passive failover** — one server handles traffic, the other is on standby
- **Priority** — determines which server owns the VIP; higher priority wins
- **Heartbeat** — regular signal sent between servers to confirm the primary is alive
- **Failback** — when the primary recovers and automatically reclaims the VIP

---

## Config Files

- [configs/primary-keepalived.conf](configs/primary-keepalived.conf) — Keepalived config for Ubuntu (Primary)
- [configs/backup-keepalived.conf](configs/backup-keepalived.conf) — Keepalived config for Kali (Backup)

---

## Tools Used

- Keepalived
- VRRP protocol
- VirtualBox
- Ubuntu 24.04
- Kali Linux
