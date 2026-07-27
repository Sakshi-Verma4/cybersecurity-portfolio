# Multi-Stage Attack Simulation & Investigation

**SOC Home Lab Project | Nmap | Hydra | Linux Log Forensics**

---

## Project Overview

This project simulates a realistic attack sequence, reconnaissance, an access attempt, and a post-exploitation action, then investigates
the full chain using log evidence rather than treating each stage as an isolated event.

## Lab Environment

| Component | Details |
|---|---|
| **Attacker Machine** | Kali Linux (VirtualBox) |
| **Target Machine** | Ubuntu Server (VirtualBox) |
| **Network** | Internal Network, 192.168.56.0/24 |
| **Target Host** | 192.168.56.103 |
| **Tools Used** | Nmap, Hydra, Linux auth.log |

---

## Investigation Objectives

1. Identify what services are exposed on the target
2. Attempt to gain access via password guessing
3. Simulate a post-exploitation persistence action
4. Investigate the resulting log evidence and determine what actually happened, versus what could be assumed

---

## Analysis Methodology

### Step 1. Reconnaissance

```
nmap -sV -Pn 192.168.56.103
```

**Key findings:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.3p1 Ubuntu 1ubuntu3
Not shown: 999 filtered tcp ports (no-response)
```
- SSH was the only accessible service, every other port was actively filtered
- `-Pn` was required since the target's own firewall (configured in an earlier project) was silently blocking Nmap's default host-discovery probe,
  even though the host responded fine to plain ping

---

### Step 2. Initial Access Attempt

```
hydra -l ubuntu -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.103
```

**Key findings:**
```
[STATUS] 144.00 tries/min, 144 tries in 00:01h, 16 active
```
- An automated password-guessing attempt was run against the SSH service found in Step 1
- No "Accepted password" entry appeared anywhere in the logs, the attempt did not succeed

---

### Step 3. Post Exploitation (simulated)

```
sudo useradd -m backdoor_user
sudo passwd backdoor_user
```

**Key findings:**
```
useradd[6143]: new group: name=backdoor_user, GID=1001
useradd[6143]: new user: name=backdoor_user, UID=1001, GID=1001, home=/home/backdoor_user, shell=/bin/sh, from=/dev/pts/1
passwd[6167]: pam_unix(passwd:chauthtok): password changed for backdoor_user
```
- A new local account was created to demonstrate what a persistence/backdoor action looks like in the logs
- This was performed directly using the existing `ubuntu` account's own admin access, not as a result of Step 2 succeeding

**Investigation note:** the first attempt to create this account produced no log entry, and a check of `/etc/passwd` confirmed the account didn't
actually exist yet. Re-running the command a second time worked, and the creation event was then found in `auth.log`. A reminder to verify an action
actually succeeded rather than assume it did just because no error appeared.

---

## Timeline

Testing was done across separate sessions with a VM restart in between, so these reflect distinct phases rather than one continuous attack:

| Time | Stage |
|---|---|
| 02:03 | Reconnaissance (Nmap) |
| 02:55 | Initial access attempt (Hydra) |
| 03:47 | Backdoor account created |
| 03:49 | Password set for backdoor account |

---

## Evidence Summary

| Evidence | Source | Confirms |
|---|---|---|
| `22/tcp open ssh OpenSSH 9.3p1` | Nmap scan | Only exposed service on target |
| `144 tries in 00:01h` | Hydra output | Automated login attempt occurred |
| `new user: name=backdoor_user` | auth.log | Account creation, with timestamp and terminal session |
| No "Accepted password" line | auth.log | Brute-force attempt did not succeed |

---

## Key Findings Summary

- **Exposed service:** SSH only (port 22), all else filtered
- **Attack attempt:** automated brute-force via Hydra, unsuccessful
- **Post-exploitation:** backdoor account created using existing legitimate access, not via a successful breach
- **Verdict:** attack attempted, not a full compromise. No unauthorized access was gained at any point in this simulation

---

## Tools Used

| Tool | Purpose |
|---|---|
| Nmap | Service/version reconnaissance |
| Hydra | Automated password-guessing attempt |
| Linux auth.log | Evidence source for all three stages |

---

## Skills Demonstrated

- ✅ Service reconnaissance and firewall-aware scanning (`-Pn`)
- ✅ Automated brute-force simulation and result verification
- ✅ Persistence/backdoor account simulation
- ✅ Cross-log investigation and evidence verification (not assuming success)
- ✅ Accurate incident assessment, distinguishing an attack attempt from an actual compromise
- ✅ Formal incident report writing
