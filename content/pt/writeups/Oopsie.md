---
title: "Writeup Hack The Box: Oopsie"
date: 2025-12-15
description: "Avaliação de segurança de uma aplicação web envolvendo Broken Access Control (IDOR) e SUID Path Hijacking."
tags: ["htb", "idor", "suid", "path-hijacking", "mitre-attack"]
layout: "page"
showPagination: true
showDate: true
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Sumário Executivo

O host **Oopsie** foi comprometido através de falhas de controle de acesso na aplicação web, permitindo bypass de autorização e upload de arquivo malicioso, resultando em Execução Remota de Código (RCE). A escalada de privilégios foi obtida explorando **PATH hijacking** em um binário **SUID root** mal implementado, culminando em acesso total ao sistema.

### Cadeia de Ataque (Mapeamento PTES)

#### 1. Descoberta e Reconhecimento
O mapeamento de rede identificou SSH e um servidor web Apache.
```bash
sudo nmap -sS -T4 -p- -sV -O --min-rate 5000 10.129.7.9
```
* **Técnica MITRE:** [T1595](https://attack.mitre.org/techniques/T1595/) - Varredura Ativa.

#### 2. Enumeração e Análise
A enumeração web revelou um endpoint oculto em `/cdn-cgi/login/`. A análise da sessão autenticada identificou uma vulnerabilidade de **Broken Access Control (IDOR)**. Ao modificar o parâmetro `id` na URL, dados administrativos (ID 34322) foram expostos.
* **Técnica MITRE:** [T1592](https://attack.mitre.org/techniques/T1592/) - Coleta de Informações do Host Vítima.

#### 3. Exploração (Acesso Inicial)
Utilizando o acesso administrativo obtido via manipulação de requisições, a página de upload tornou-se acessível. Um arquivo **.phtml** contendo um payload reverse TCP foi enviado, garantindo uma shell remota.
* **Técnica MITRE:** [T1505.003](https://attack.mitre.org/techniques/T1505/003/) - Componente de Software de Servidor: Web Shell.

#### 4. Pós-Exploração e Escalação de Privilégios
A enumeração local revelou credenciais em `db.php` (`robert:M3g4C0rpUs3r!`), permitindo movimentação lateral para o usuário `robert`. 

Análises posteriores identificaram um binário SUID em `/usr/bin/bugtracker` que executava o comando `cat` sem um caminho absoluto. Isso foi explorado via **PATH Hijacking**:
```bash
export PATH=/tmp/privesc:$PATH
/usr/bin/bugtracker
```
* **Técnica MITRE:** [T1548.001](https://attack.mitre.org/techniques/T1548/001/) - Abuso de Mecanismo de Controle de Elevação: Setuid e Setgid.

### Remediação (NIST SP 800-115)

* **Controle de Acesso:** Implementar verificações de autorização no lado do servidor e validar permissões independentemente de parâmetros do lado do cliente.
* **Desenvolvimento Seguro:** Utilizar caminhos absolutos em binários SUID e sanitizar variáveis de ambiente como `PATH`.
* **Higiene de Credenciais:** Evitar o armazenamento de credenciais em texto claro em arquivos de configuração.