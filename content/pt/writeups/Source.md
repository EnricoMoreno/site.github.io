---
title: "TryHackMe Writeup: Source"
date: 2025-12-11
description: "Exploração de backdoor público conhecido no Webmin 1.890 resultando em Execução Remota de Código não autenticada como root."
tags: ["tryhackme", "webmin", "cve", "rce", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Resumo Executivo

Uma avaliação de segurança direcionada identificou uma vulnerabilidade crítica na infraestrutura do servidor. O host estava executando a versão 1.890 do Webmin, que é conhecida por conter um backdoor de supply-chain divulgado publicamente. A exploração dessa falha permitiu a Execução Remota de Código (RCE) não autenticada, concedendo imediatamente privilégios de nível root e resultando no comprometimento total do sistema.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Reconhecimento
A varredura completa das portas TCP identificou os serviços SSH e HTTP executando o MiniServ 1.890 (Webmin) na porta 10000.
```bash
nmap -sS -sV -Pn -O -p- -T4 10.67.170.253
```
* **Técnica MITRE:** [T1595](https://attack.mitre.org/techniques/T1595/) - Active Scanning.

#### 2. Análise de Vulnerabilidades
O fingerprint de versão confirmou o Webmin 1.890. Esta versão específica foi comprometida em nível de código-fonte (ataque de supply chain), introduzindo um backdoor que permite a execução de comandos arbitrários via endpoint `password_change.cgi`.
* **Técnica MITRE:** [T1195.002](https://attack.mitre.org/techniques/T1195/002/) - Supply Chain Compromise: Compromise Software Supply Chain (Historical Context).

#### 3. Exploração & Elevação de Privilégio
A vulnerabilidade foi aproveitada utilizando o módulo `linux/http/webmin_backdoor` do framework Metasploit. O payload foi executado com sucesso, ignorando a autenticação.
```bash
use linux/http/webmin_backdoor
set RHOSTS 10.67.170.253
set RPORT 10000
run
```
* **Técnica MITRE:** [T1190](https://attack.mitre.org/techniques/T1190/) - Exploit Public-Facing Application.

Como o Webmin executa com privilégios de root por padrão, o reverse shell resultante herdou essas permissões de forma nativa, não exigindo etapas adicionais de elevação de privilégios.
```bash
# Validação pós-exploração
whoami # root
```

### Recomendações (NIST SP 800-115)

* **Aplicação Imediata de Patch:** Atualizar o Webmin para uma versão segura e verificada (≥ 1.930). Verificar as assinaturas GPG dos pacotes baixados para prevenir vulnerabilidades de supply-chain.
* **Controle de Acesso:** Restringir o acesso à interface de gerenciamento do Webmin (porta 10000) utilizando regras de firewall (`ufw` ou `iptables`). O acesso deve ser permitido apenas a partir de faixas de IP administrativos confiáveis.
* **Hardening de Serviço:** Configurar o Webmin para escutar exclusivamente na interface localhost (`bind=127.0.0.1` em `miniserv.conf`) e exigir que os administradores o acessem de forma segura via tunelamento SSH.
* **Resposta a Incidentes:** Dada a natureza do backdoor, conduzir uma revisão forense completa em busca de mecanismos de persistência (ex: crontabs maliciosos, arquivos alterados em `/etc/systemd/system/`) e rotacionar todas as senhas de sistema e chaves SSH.