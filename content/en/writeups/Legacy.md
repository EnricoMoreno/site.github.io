---
title: "Hack The Box Writeup: Legacy"
date: 2025-12-18
description: "Exploitation of SMBv1 (MS08-067) for direct SYSTEM access on Windows XP."
tags: ["htb", "smb", "ms08-067", "legacy", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

The **Legacy** machine was compromised by exploiting a critical vulnerability in **SMBv1 (MS08-067 / CVE-2008-4250)**. The exploit allowed for unauthenticated Remote Code Execution and immediate access as **NT AUTHORITY\SYSTEM**.

### Attack Chain (PTES Mapping)

#### 1. Discovery & Reconnaissance
Scanning identified active NetBIOS and SMB services on a Windows XP host.
```bash
sudo nmap -sS -T4 -p- -sV -Pn --min-rate 5000 10.129.10.40
```
* **MITRE Technique:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Analysis
Protocol enumeration confirmed the presence of **SMBv1**, which is notoriously vulnerable to the MS08-067 NetAPI exploit.
* **MITRE Technique:** [T1210](https://attack.mitre.org/techniques/T1210/) - Exploitation of Remote Services.

#### 3. Exploitation
The Metasploit module `exploit/windows/smb/ms08_067_netapi` was used to gain a Meterpreter session with maximum privileges. No further escalation was required.
* **MITRE Technique:** [T1133](https://attack.mitre.org/techniques/T1133/) - External Remote Services.

### Remediation (NIST SP 800-115)

* **Protocol Security:** Disable **SMBv1** immediately across the network.
* **System Migration:** Retire legacy Windows XP systems in favor of supported operating systems.
* **Access Control:** Restrict SMB ports (139/445) via hardware firewalls.