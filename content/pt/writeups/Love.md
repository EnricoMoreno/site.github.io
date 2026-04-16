---
title: "Writeup Hack The Box: Love"
date: 2025-12-18
description: "Cadeia de SSRF, vazamento de credenciais e escalação de privilégios AlwaysInstallElevated."
tags: ["htb", "ssrf", "alwaysinstallelevated", "windows", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

**Love** foi comprometida explorando uma falha de **SSRF** em um serviço de staging interno que expôs credenciais administrativas. Estas foram usadas para ganhar uma shell reversa via uma aplicação web vulnerável. Os privilégios foram então escalados para **SYSTEM** através da política **AlwaysInstallElevated**.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Enumeração e SSRF
Descoberta de um ambiente de staging (`staging.love.htb`). Uma funcionalidade "Demo" permitia o envio de URLs, o que foi usado para realizar um ataque **SSRF** contra `127.0.0.1:5000`, vazando a senha do admin: `@LoveIsInTheAir!!!!`.
* **Técnica MITRE:** [T1557](https://attack.mitre.org/techniques/T1557/) - Adversário no Meio (AiTM).

#### 2. Exploração
As credenciais foram usadas para logar em uma aplicação de "Sistema de Votação". Um exploit foi usado para subir uma shell PHP, garantindo acesso inicial como um usuário de baixo privilégio.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Componente de Software de Servidor: Web Shell.

#### 3. Escalação de Privilégios
A enumeração identificou que as chaves de registro **AlwaysInstallElevated** estavam definidas como 1. Um payload **.msi** malicioso foi criado e executado via `msiexec`, garantindo uma shell SYSTEM.
```bash
msiexec /quiet /qn /i C:\Administration\payload.msi
```
* **Técnica MITRE:** [T1548.002](https://attack.mitre.org/techniques/T1548/002/) - Abuso de Mecanismo de Controle de Elevação: Bypass de Controle de Conta de Usuário (UAC).

### Remediação (NIST SP 800-115)

* **Segurança Web:** Prevenir SSRF validando e criando uma whitelist de URLs permitidas.
* **Proteção de Credenciais:** Garantir que serviços internos não vazem credenciais em respostas.
* **Hardening de SO:** Desabilitar a política **AlwaysInstallElevated**.