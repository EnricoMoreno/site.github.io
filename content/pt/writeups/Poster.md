---
title: "TryHackMe Writeup: Poster"
date: 2025-12-01
description: "Comprometimento de banco de dados via credenciais padrão do PostgreSQL, enumeração interna e elevação de privilégio através de sudo irrestrito."
tags: ["tryhackme", "postgresql", "sudo", "privesc", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Avaliação técnica de segurança conduzida em alinhamento com o framework NIST SP 800-115. Uma instância PostgreSQL exposta com credenciais padrão permitiu acesso não autenticado ao banco de dados e Execução Remota de Código (RCE). A pós-exploração revelou credenciais sensíveis do sistema na raiz do servidor web, levando ao acesso SSH. O comprometimento total do sistema foi alcançado abusando de privilégios irrestritos de `sudo`.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento & Enumeração
A varredura de portas identificou um serviço PostgreSQL exposto (porta 5432) juntamente com HTTP e SSH.
```bash
nmap -sS -sV -Pn -O -p- -T4 10.64.154.252
```
* **Técnica MITRE:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Análise de Vulnerabilidades & Exploração
O banco de dados PostgreSQL foi autenticado utilizando credenciais padrão (`postgres:password`) via Metasploit.
```bash
use auxiliary/scanner/postgres/postgres_login
```
* **Técnica MITRE:** [T1078.001](https://attack.mitre.org/techniques/T1078/001/) - Valid Accounts: Default Accounts.

A exploração subsequente utilizou o recurso `COPY FROM PROGRAM` do PostgreSQL para executar comandos de sistema e estabelecer um reverse shell.
```bash
use exploit/multi/postgres/postgres_copy_from_program_cmd_exec
```
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Movimentação Lateral
A enumeração interna revelou um arquivo de configuração (`/var/www/html/config.php`) contendo credenciais em texto claro (`alison:p4ssw0rdS3cur3!#`), que foram reutilizadas para estabelecer uma sessão SSH.
* **Técnica MITRE:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.
* **Técnica MITRE:** [T1021.004](https://attack.mitre.org/techniques/T1021/004/) - Remote Services: SSH.

#### 4. Elevação de Privilégio
O usuário `alison` possuía acesso `sudo` irrestrito, permitindo a elevação instantânea para root.
```bash
sudo -l # (ALL : ALL) ALL
sudo /bin/bash
```
* **Técnica MITRE:** [T1548.003](https://attack.mitre.org/techniques/T1548/003/) - Abuse Elevation Control Mechanism: Sudo and Sudo Caching.

### Recomendações (NIST SP 800-115)

* **Segurança do Banco de Dados:** Alterar as credenciais padrão do banco de dados imediatamente. Vincular o serviço PostgreSQL exclusivamente ao `localhost` (`listen_addresses = 'localhost'`) no arquivo `postgresql.conf` e restringir o acesso via `pg_hba.conf`.
* **Gestão de Credenciais:** Remover credenciais em texto claro de diretórios públicos da web. Implementar gestão segura de segredos (ex: Variáveis de Ambiente ou HashiCorp Vault).
* **Menor Privilégio:** Revogar os privilégios irrestritos de `sudo` para o usuário `alison`. Implementar controles de acesso rigorosos e granulares via `visudo`.