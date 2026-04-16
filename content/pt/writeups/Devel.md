---
title: "Writeup Hack The Box: Devel"
date: 2025-12-17
description: "Exploração de FTP e Escalação de Privilégios Windows via MS10-015 (KiTrap0D)."
tags: ["htb", "ftp", "iis", "ms10-015", "privesc", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

A máquina **Devel** apresenta um serviço FTP com login anônimo e permissões de escrita, permitindo o upload de um payload **ASPX** que leva ao RCE via IIS 7.5. A escalada de privilégios foi obtida utilizando a vulnerabilidade **MS10-015 (KiTrap0D)**, resultando em acesso de nível **SYSTEM**.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta e Enumeração
Identificação de FTP (porta 21) e HTTP (porta 80). O login anônimo estava habilitado e o diretório raiz do FTP estava mapeado para a webroot do IIS.
```bash
nmap -Pn -sS -p- -T4 -sV 10.129.9.61
```
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploração de Aplicação Voltada para o Público.

#### 2. Exploração (Acesso Inicial)
Um payload ASPX gerado pelo `msfvenom` foi enviado via FTP. O acesso ao arquivo pelo navegador disparou a shell reversa.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Componente de Software de Servidor: Web Shell.

#### 3. Escalação de Privilégios
A enumeração local identificou o sistema como vulnerável ao **MS10-015**. O exploit foi executado para elevar a sessão para `NT AUTHORITY\SYSTEM`.
```text
meterpreter > use exploit/windows/local/ms10_015_kitrap0d
meterpreter > getuid # NT AUTHORITY\SYSTEM
```
* **Técnica MITRE:** [T1068](https://attack.mitre.org/techniques/T1068/) - Exploração para Escalação de Privilégios.

### Remediação (NIST SP 800-115)

* **Hardening de Configuração:** Desabilitar o login FTP anônimo e remover permissões de escrita da webroot.
* **Gestão de Vulnerabilidades:** Atualizar o SO Windows legado e aplicar patches de segurança críticos.