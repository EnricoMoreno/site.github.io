---
title: "TryHackMe Writeup: Poster"
date: 2025-12-01
description: "Database compromise via default PostgreSQL credentials, internal enumeration, and privilege escalation through unrestricted sudo."
tags: ["tryhackme", "postgresql", "sudo", "privesc", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Technical security assessment conducted in alignment with the NIST SP 800-115 framework. An exposed PostgreSQL instance with default credentials allowed unauthenticated database access and Remote Code Execution (RCE). Post-exploitation revealed sensitive system credentials in the web root, leading to SSH access. Full system compromise was achieved by abusing unrestricted `sudo` privileges.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance & Enumeration
Port scanning identified an exposed PostgreSQL service (port 5432) alongside HTTP and SSH.
```bash
nmap -sS -sV -Pn -O -p- -T4 10.64.154.252
```
* **MITRE Technique:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Vulnerability Analysis & Exploitation
The PostgreSQL database was authenticated using default credentials (`postgres:password`) via Metasploit.
```bash
use auxiliary/scanner/postgres/postgres_login
```
* **MITRE Technique:** [T1078.001](https://attack.mitre.org/techniques/T1078/001/) - Valid Accounts: Default Accounts.

Subsequent exploitation utilized PostgreSQL's `COPY FROM PROGRAM` feature to execute system commands and establish a reverse shell.
```bash
use exploit/multi/postgres/postgres_copy_from_program_cmd_exec
```
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Lateral Movement
Internal enumeration revealed a configuration file (`/var/www/html/config.php`) containing plaintext credentials (`alison:p4ssw0rdS3cur3!#`), which were reused to establish an SSH session.
* **MITRE Technique:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.
* **MITRE Technique:** [T1021.004](https://attack.mitre.org/techniques/T1021/004/) - Remote Services: SSH.

#### 4. Privilege Escalation
The user `alison` possessed unrestricted `sudo` access, allowing instant elevation to root.
```bash
sudo -l # (ALL : ALL) ALL
sudo /bin/bash
```
* **MITRE Technique:** [T1548.003](https://attack.mitre.org/techniques/T1548/003/) - Abuse Elevation Control Mechanism: Sudo and Sudo Caching.

### Recommendations (NIST SP 800-115)

* **Database Security:** Change default database credentials immediately. Bind the PostgreSQL service exclusively to `localhost` (`listen_addresses = 'localhost'`) in `postgresql.conf` and restrict access via `pg_hba.conf`.
* **Credential Management:** Remove plaintext credentials from web root directories. Implement secure secret management (e.g., Environment Variables or HashiCorp Vault).
* **Least Privilege:** Revoke unrestricted `sudo` privileges for the user `alison`. Implement strict, granular access controls via `visudo`.