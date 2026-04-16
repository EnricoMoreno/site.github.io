---
title: "Writeup Hack The Box: Lame"
date: 2025-12-18
description: "Exploração do Samba CVE-2007-2447 (usermap_script) para acesso root instantâneo."
tags: ["htb", "samba", "rce", "linux", "mitre-attack"]
---

### Sumário Executivo

A máquina **Lame** foi comprometida através de uma vulnerabilidade conhecida no **Samba (CVE-2007-2447)**. O acesso inicial foi obtido diretamente com privilégios administrativos, resultando na captura imediata das flags de usuário e root.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta
O Nmap identificou serviços FTP, SSH e Samba.
```bash
sudo nmap -sS -T4 -p- -sV -Pn 10.129.10.2
```
* **Técnica MITRE:** [T1046](https://attack.mitre.org/techniques/T1046/) - Varredura de Serviços de Rede.

#### 2. Enumeração
A captura de banners de serviço identificou o **Samba 3.0.20-Debian**, que é vulnerável ao RCE `usermap_script`.
* **Técnica MITRE:** [T1210](https://attack.mitre.org/techniques/T1210/) - Exploração de Serviços Remotos.

#### 3. Exploração
Utilizando o módulo `multi/samba/usermap_script`, uma shell de comando foi estabelecida. Como a vulnerabilidade permite a execução como o usuário que roda o serviço (root), nenhuma escalação de privilégios foi necessária.
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploração de Aplicação Voltada para o Público.

### Remediação (NIST SP 800-115)

* **Gestão de Patches:** Atualizar o Samba para uma versão segura e corrigida.
* **Hardening de Serviço:** Desabilitar o acesso SMB anônimo e restringir o tráfego para hosts internos autorizados.