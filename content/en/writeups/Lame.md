---
title: "Hack The Box Writeup: Lame"
date: 2025-12-18
description: "Exploitation of Samba CVE-2007-2447 (usermap_script) for instant root access."
tags: ["htb", "samba", "rce", "linux", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

The **Lame** machine was compromised through a well-known vulnerability in **Samba (CVE-2007-2447)**. Initial access was obtained directly with administrative privileges, resulting in the immediate capture of both user and root flags.

### Attack Chain (PTES Mapping)

#### 1. Discovery
Nmap identified FTP, SSH, and Samba services.
```bash
sudo nmap -sS -T4 -p- -sV -Pn 10.129.10.2
```
* **MITRE Technique:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Enumeration
Service banner grabbing identified **Samba 3.0.20-Debian**, which is vulnerable to the `usermap_script` RCE.
* **MITRE Technique:** [T1210](https://attack.mitre.org/techniques/T1210/) - Exploitation of Remote Services.

#### 3. Exploitation
Using the `multi/samba/usermap_script` module, a command shell was established. Because the vulnerability allows execution as the user running the service (root), no privilege escalation was necessary.
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

### Remediation (NIST SP 800-115)

* **Patch Management:** Update Samba to a secure, patched version.
* **Service Hardening:** Disable anonymous SMB access and restrict traffic to authorized internal hosts.