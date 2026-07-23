# SSH Bruteforce Investigation #2 — Defense Mechanism Analysis

## Objective
Simulate a second brute-force attack against SSH, investigate it through log analysis, and evaluate the effectiveness of the target's built-in defenses
going beyond simple detection to assess why an attack succeeded or failed.

## Attack Simulation
Used Hydra from Kali to attempt password guessing against Ubuntu's SSH service.

```bash
hydra -l ubuntu -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.103
```

## Investigation
```bash
sudo grep "sshd" /var/log/auth.log | tail -30
```

**Findings:**
- Every login attempt originated from a single source IP: `192.168.56.102`
- Multiple connection attempts occurred within the same second, confirming automated (non-human) activity
- Each connection was terminated after 6 failed password attempts, triggered by SSH's built-in `MaxAuthTries` setting
- No `"Accepted password"` entry exists anywhere in the log — the attack did not succeed

## Key Technical Detail: Connections vs. Attempts
Each failed connection (identified by a unique client-side port number, e.g.56750, 56828) represents one distinct login session — not a different service
or port on the target. All connections targeted the same destination: `192.168.56.103:22`. Once a connection hit the 6-attempt limit, Ubuntu disconnected it;
Hydra responded by immediately opening a new connection and continuing, cycling through connections rather than being stopped outright.

## Incident Report

**Summary**: Automated SSH brute-force attack detected and unsuccessful; target's connection-attempt limit prevented credential compromise.

**Timeline**: Attack began at 01:02:05, with dozens of connection attempts recorded within the same second, each terminated after 6 failed guesses.

**Evidence**: Repeated "maximum authentication attempts exceeded" and "Disconnecting authenticating user" log entries, consistent source IP
across all attempts, no successful login entry present in the log.

**Assessment**: Timestamp clustering confirms automated tool activity. MaxAuthTries successfully prevented any single connection from attempting
enough passwords to succeed. However, this defense only limits attempts *per connection* — it does not prevent an attacker from reconnecting
indefinitely, meaning it slows an attack rather than stopping it outright.

**Recommendation**: Layer additional defenses rather than relying on MaxAuthTries alone:

**SSH key-based authentication** — removes password guessing as a viable attack vector entirely, rather than slowing it down.

## What I Learned
- A defense that limits attempts *per connection* is not the same as a defense that stops an *attacker* — distinguishing these two is critical
  when assessing real security posture
- Client-side port numbers in logs represent individual connection sessions, not different services or targets on the destination machine
- Absence of an "Accepted password" log entry is the definitive evidence needed to confirm an attack failed — never assume success or failure
  without checking for that specific line
