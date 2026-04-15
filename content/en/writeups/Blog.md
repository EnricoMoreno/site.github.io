---
title: "TryHackMe Writeup: Blog"
date: 2025-11-28
description: "WordPress exploitation and privilege escalation via SUID (Environment Variable Injection)."
tags: ["tryhackme", "wordpress", "suid", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Technical security assessment conducted under the NIST SP 800-115 framework. The host running WordPress exhibited weaknesses in authentication policies, allowing RCE via malicious upload. Privilege escalation exploited a lack of environment variable sanitization in a custom binary, resulting in full system compromise.

### Attack Chain (PTES Mapping)

#### 1. Pre-engagement & Intelligence Gathering
Attack surface mapping performed via Nmap.
```bash
sudo nmap -Pn -sS -sV -T4 -p- 10.64.131.151
```
* **MITRE Technique:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Vulnerability Analysis
CMS enumeration identified valid users `bjoel` and `kwheel`. A critical vulnerability was found in the password policy.
```bash
wpscan --url [http://10.64.131.151](http://10.64.131.151) --usernames kwheel --passwords rockyou.txt
```
* **MITRE Technique:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Brute Force: Password Cracking.

#### 3. Exploitation (Initial Access)
Using the discovered credentials `kwheel:cutiepie1`, a web shell was successfully uploaded to the server.
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 4. Post-Exploitation (Privilege Escalation)
A custom SUID binary located at `/usr/sbin/checker` was identified. Exploitation involved environment variable injection, allowing arbitrary code execution as root.
```bash
export admin=1 && /usr/sbin/checker
```
* **MITRE Technique:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recommendations (NIST SP 800-115)

* **Access Control:** Implement MFA and account lockout policies for WordPress.
* **System Integrity:** Remove SUID bits from binaries that rely on unsanitized environment variables (`chmod -s /usr/sbin/checker`).