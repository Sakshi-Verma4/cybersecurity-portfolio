# SSH Brute-Force Attack Detection & Incident Report

## Objective
Simulate a real brute-force attack against SSH, then investigate it purely 
from log evidence — no prior knowledge of the attack — to practice 
incident response and log analysis.

## Attack Simulation
Used Hydra from Kali to attempt password guessing against Ubuntu's SSH service.

```bash
hydra -l ubuntu -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.103
```

## Investigation (Log Analysis)
Investigated `/var/log/auth.log` on the target to identify the attack pattern.

```bash
sudo tail -20 /var/log/auth.log
sudo grep -c "sshd" /var/log/auth.log
```

**Findings:**
- 906 total sshd-related log lines generated during the attack window
- Multiple connection attempts logged within the same second (e.g. 14 
  attempts within ~200ms at timestamp 06:42:52), confirming automated, 
  non-human activity
- Source IP consistently identified across all attempts

## Baseline Comparison
Before simulating the attack, I tested a legitimate manual login with one 
mistyped password, producing exactly 1 failed attempt followed by success 
4 seconds later — establishing a clear baseline for what normal human 
behavior looks like in the logs, versus the attack pattern above.

## Incident Report

**Summary**: Automated brute-force attack detected against SSH service, 
originating from a single source IP.

**Timeline**: Attack began at 06:42:52, with over a dozen authentication 
attempts within a single second.

**Evidence**: 906 sshd log entries generated in a short window; timestamp 
clustering (multiple attempts per second) inconsistent with manual human login.

**Assessment**: Pattern is consistent with automated password-guessing tool, 
not human error — confirmed by comparison against a known-clean baseline 
(1 human failed attempt = 4 second gap before retry).

**Recommendation**: Disable SSH password authentication in favor of 
key-based authentication to eliminate brute-force risk entirely. Implement 
fail2ban or similar rate-limiting to lock out repeated failed attempts.

## What I Learned
- Automated attacks are distinguishable from human error by timestamp 
  density, not just attempt count
- Log search terms matter — searching only for "Failed password" missed 
  attempts logged as "Connection closed by authenticating user", teaching 
  me that detection rules need broader pattern matching
- Established a real baseline before generating an attack, which made the 
  comparison meaningful rather than assumed
