---
title: "Hack The Box Writeup: Love"
date: 2025-12-18
description: "Chain of SSRF, credential leak, and AlwaysInstallElevated privilege escalation."
tags: ["htb", "ssrf", "alwaysinstallelevated", "windows", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Executive Summary

**Love** was compromised by exploiting an **SSRF** flaw in an internal staging service that exposed administrative credentials. These were used to gain a reverse shell via a vulnerable web application. Privileges were then escalated to **SYSTEM** via the **AlwaysInstallElevated** policy.

### Attack Chain (PTES Mapping)

#### 1. Enumeration & SSRF
Discovery of a staging environment (`staging.love.htb`). A "Demo" feature allowed URL submissions, which was used to perform an **SSRF** attack against `127.0.0.1:5000`, leaking the admin password: `@LoveIsInTheAir!!!!`.
* **MITRE Technique:** [T1557](https://attack.mitre.org/techniques/T1557/) - Adversary-in-the-Middle.

#### 2. Exploitation
The credentials were used to log into a "Voting System" application. An exploit was used to upload a PHP shell, granting initial access as a low-privileged user.
* **MITRE Technique:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 3. Privilege Escalation
Enumeration identified that the **AlwaysInstallElevated** registry keys were set to 1. A malicious **.msi** payload was created and executed via `msiexec`, granting a SYSTEM shell.
```bash
msiexec /quiet /qn /i C:\Administration\payload.msi
```
* **MITRE Technique:** [T1548.002](https://attack.mitre.org/techniques/T1548/002/) - Abuse Elevation Control Mechanism: Bypass User Account Control.

### Remediation (NIST SP 800-115)

* **Web Security:** Prevent SSRF by validating and whitelisting permitted URLs.
* **Credential Protection:** Ensure internal services do not leak credentials in responses.
* **OS Hardening:** Disable the **AlwaysInstallElevated** policy.