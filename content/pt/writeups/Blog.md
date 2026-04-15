---
title: "TryHackMe Writeup: Blog"
date: 2025-11-28
description: "Security assessment baseado no PTES. Exploração de WordPress e elevação de privilégio via SUID (Environment Variable Injection)."
tags: ["tryhackme", "wordpress", "suid", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Avaliação técnica de segurança conduzida sob o framework NIST SP 800-115. O host executando WordPress apresentou fragilidades nas políticas de autenticação, permitindo RCE (Remote Code Execution) via upload malicioso. A elevação de privilégios explorou a falta de sanitização de variáveis de ambiente em um binário customizado, resultando no comprometimento total do sistema.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Pré-engajamento & Coleta de Informações
Mapeamento da superfície de ataque realizado via Nmap.
```bash
sudo nmap -Pn -sS -sV -T4 -p- 10.64.131.151
```
* **Técnica MITRE:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Análise de Vulnerabilidades
A enumeração do CMS identificou os usuários válidos `bjoel` e `kwheel`. Uma vulnerabilidade crítica foi encontrada na política de senhas.
```bash
wpscan --url [http://10.64.131.151](http://10.64.131.151) --usernames kwheel --passwords rockyou.txt
```
* **Técnica MITRE:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Brute Force: Password Cracking.

#### 3. Exploração (Acesso Inicial)
Utilizando as credenciais descobertas `kwheel:cutiepie1`, um web shell foi carregado com sucesso no servidor.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 4. Pós-Exploração (Elevação de Privilégio)
Foi identificado um binário SUID customizado localizado em `/usr/sbin/checker`. A exploração envolveu a injeção de variável de ambiente, permitindo a execução arbitrária de código como root.
```bash
export admin=1 && /usr/sbin/checker
```
* **Técnica MITRE:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recomendações (NIST SP 800-115)

* **Controle de Acesso:** Implementar MFA e políticas de bloqueio de conta (lockout) para o WordPress.
* **Integridade do Sistema:** Remover os bits SUID de binários que dependem de variáveis de ambiente não sanitizadas (`chmod -s /usr/sbin/checker`).