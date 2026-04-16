---
title: "Hack The Box Writeup: Oopsie"
date: 2025-12-15
description: "Security assessment of a web application involving Broken Access Control (IDOR) and SUID Path Hijacking."
tags: ["htb", "idor", "suid", "path-hijacking", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

The **Oopsie** host was compromised through access control flaws in the web application, allowing for authorization bypass and malicious file upload, resulting in Remote Code Execution (RCE). Privilege escalation was achieved by exploiting **PATH hijacking** on a poorly implemented **SUID root** binary, culminating in full system access.

### Attack Chain (PTES Mapping)

#### 1. Discovery & Reconnaissance
Network mapping identified SSH and an Apache web server.
```bash
sudo nmap -sS -T4 -p- -sV -O --min-rate 5000 10.129.7.9
```
* **MITRE Technique:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Enumeration & Analysis
Web enumeration revealed a hidden endpoint at `/cdn-cgi/login/`. Analysis of the authenticated session identified a **Broken Access Control (IDOR)** vulnerability. By modifying the `id` parameter in the URL, administrative data (ID 34322) was exposed.
* **MITRE Technique:** [T1592](https://attack.mitre.org/techniques/T1592/) - Gather Victim Host Information.

#### 3. Exploitation (Initial Access)
Using the administrative access gained via request manipulation, the upload page became accessible. A **.phtml** file containing a reverse TCP payload was uploaded, granting a remote shell.
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 4. Post-Exploitation & Privilege Escalation
Local enumeration revealed credentials in `db.php` (`robert:M3g4C0rpUs3r!`), allowing lateral movement to the user `robert`. 

Further analysis identified an SUID binary at `/usr/bin/bugtracker` that executed the `cat` command without an absolute path. This was exploited via **PATH Hijacking**:
```bash
export PATH=/tmp/privesc:$PATH
/usr/bin/bugtracker
```
* **MITRE Technique:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Remediation (NIST SP 800-115)

* **Access Control:** Implement server-side authorization checks and validate permissions independently of client-side parameters.
* **Secure Development:** Use absolute paths in SUID binaries and sanitize environment variables like `PATH`.
* **Credential Hygiene:** Avoid storing plaintext credentials in configuration files.