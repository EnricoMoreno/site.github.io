---
title: "Writeup Hack The Box: Legacy"
date: 2025-12-18
description: "Exploração de SMBv1 (MS08-067) para acesso direto como SYSTEM no Windows XP."
tags: ["htb", "smb", "ms08-067", "legacy", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

A máquina **Legacy** foi comprometida explorando uma vulnerabilidade crítica no **SMBv1 (MS08-067 / CVE-2008-4250)**. O exploit permitiu a Execução Remota de Código sem autenticação e acesso imediato como **NT AUTHORITY\SYSTEM**.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta e Reconhecimento
A varredura identificou serviços NetBIOS e SMB ativos em um host Windows XP.
```bash
sudo nmap -sS -T4 -p- -sV -Pn --min-rate 5000 10.129.10.40
```
* **Técnica MITRE:** [T1046](https://attack.mitre.org/techniques/T1046/) - Varredura de Serviços de Rede.

#### 2. Análise
A enumeração de protocolo confirmou a presença de **SMBv1**, que é notoriamente vulnerável ao exploit MS08-067 NetAPI.
* **Técnica MITRE:** [T1210](https://attack.mitre.org/techniques/T1210/) - Exploração de Serviços Remotos.

#### 3. Exploração
O módulo do Metasploit `exploit/windows/smb/ms08_067_netapi` foi utilizado para obter uma sessão Meterpreter com privilégios máximos. Nenhuma escalada adicional foi necessária.
* **Técnica MITRE:** [T1133](https://attack.mitre.org/techniques/T1133/) - Serviços Remotos Externos.

### Remediação (NIST SP 800-115)

* **Segurança de Protocolo:** Desabilitar o **SMBv1** imediatamente em toda a rede.
* **Migração de Sistema:** Aposentar sistemas Windows XP legados em favor de sistemas operacionais suportados.
* **Controle de Acesso:** Restringir portas SMB (139/445) via firewalls de hardware.