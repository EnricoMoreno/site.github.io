---
title: "Guia para o eJPT"
layout: "page"
showPagination: false
showDate: true
date: 2026-04-03
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Introdução

Nada mais justo do que iniciar as postagens neste site com um guia para a prova **eJPT (eLearnSecurity Junior Penetration Tester)**, sem spoilers. Realizei o curso preparatório da **INE**; se você possui condições, o investimento é muito interessante para compreender a metodologia da instituição, embora os temas estejam amplamente disponíveis online.

### Estrutura do Exame

Antes dos conteúdos, é necessário entender a mecânica da prova:
* **Tempo:** 48 horas de duração.
* **Formato:** Questões de múltipla escolha que dependem de informações encontradas durante a invasão e keys/flags.
* **Ambiente:** Você utiliza uma **máquina virtual (VM)** no navegador fornecida por eles, não a sua VM ou PC. 

> **Nota técnica:** O ping pode ser elevado para usuários no Brasil (como foi no meu caso), o que pode acabar dificultando um pouco.

### O que você precisa saber?

Para ir bem na prova, é fundamental estar confortável com os seguintes pilares:

1.  **Redes:** Conceitos básicos de **TCP/IP** e o modelo OSI.
2.  **Sistemas Operacionais:** Navegação em **Linux** e **Windows**, localização de arquivos sensíveis e permissões.
3.  **Serviços Comuns:** Exploração e enumeração de **SSH, HTTP, MySQL, SMB e FTP**.
4.  **Ferramentas:** Domínio do **Nmap** (essencial) e do **Metasploit Framework** (auxiliar, mas não obrigatório).
5.  **Técnicas:** Escalação de privilégios simples, **Pivoting** (movimentação entre redes) e, principalmente, **Enumeração**.

> **Dica de Ouro:** A enumeração é a fase mais crítica. Se estiver travado, enumere novamente. A maioria dos erros e esses momentos onde não sabemos o que fazer, acontece por ignorar um serviço ou porta aberta.

### Recomendações e Metodologia

* **Pesquisa Liberada:** Você pode consultar documentações e o Google durante a prova. O que é proibido é a colaboração externa (ou mais famoso, colar).
* **Gestão de Tempo:** Com 48 horas, não há necessidade de pressa. Se você se sentir cansado ou bloqueio mental, sai do computador e volte mais tarde.
* **Armadilhas de Dicionário:** Evite o uso de wordlists massivas como a `rockyou.txt` em conexões lentas, a menos que seja estritamente necessário. A VM do exame já disponibiliza listas otimizadas para o cenário.

### Laboratórios de Prática (CTFs)

As máquinas abaixo são excelentes para fixar os conceitos exigidos no eJPT. Algumas podem exigir assinatura VIP nas plataformas quando você estiver lendo isso (mas vale a pena). Clique no link das máquinas para acessar o write-up que fiz para elas.:

| Plataforma | Máquina / Laboratório |
| :--- | :--- |
| **TryHackMe** | [Ignite](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-ignite-easy.md), [RootMe](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-rootme-easy.md), [Startup](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-startup-easy.md), [Blog](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-blog-medium.md), [Anonymous](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-anonymous-medium.md), [Internal](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-internal-hard.md), [Poster](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-poster-easy.md), [Source](https://github.com/EnricoMoreno/ctf-writeups/blob/main/THM/thm-source-easy.md) |
| **HackTheBox** | [Oopsie](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-oopsie-veryeasy.md), [Pennyworth](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-pennyworth-veryeasy.md), [Devel](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-devel-easy.md), [Legacy](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-legacy-easy.md), [Lame](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-lame-easy.md), [Love](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-love-easy.md), [Jeeves](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-jeeves-medium.md), [Hawk](https://github.com/EnricoMoreno/ctf-writeups/blob/main/HTB/htb-hawk-medium.md) |
---
Se você conseguir resolver as máquinas acima sem auxílio constante de writeups, a prova não apresentará grandes dificuldades técnicas. O maior desafio costuma ser a pressão psicológica e a atenção aos detalhes na fase de coleta de informações.


### Resumo da Ópera 

> O **eJPT** foi uma prova e experiência ótima para o começo nessa área que pode ser tão profunda. Se você tiver condições para fazê-la, eu recomendo muito, ainda mais se conseguir aproveitar alguma **promoção**, o que torna o custo-benefício ainda melhor. Ela oferece uma **base excelente** para você iniciar sua jornada no mundo da **segurança ofensiva**.