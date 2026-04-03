---
title: "eJPT Guide"
layout: "page"
showPagination: false
showDate: true
date: 2026-04-03
showReadingTime: true
showWordCount: true
showAuthorBottom: true
---

### Introduction

It is only fair to start the posts on this site with a guide for the **eJPT (eLearnSecurity Junior Penetration Tester)** exam, without spoilers. I took the **INE** preparatory course; if you have the means, the investment is very interesting to understand the institution's methodology, although the topics are widely available online.

### Exam Structure

Before the content, it is necessary to understand the exam mechanics:
* **Time:** 48 hours duration.
* **Format:** Multiple-choice questions that depend on information found during the intrusion and keys/flags.
* **Environment:** You use a browser-based **virtual machine (VM)** provided by them, not your own VM or PC. 

> **Technical note:** The ping can be high for users in Brazil (as it was in my case), which can end up making it a bit difficult.

### What do you need to know?

To do well on the exam, it is fundamental to be comfortable with the following pillars:

1.  **Networking:** Basic concepts of **TCP/IP** and the OSI model.
2.  **Operating Systems:** **Linux** and **Windows** navigation, sensitive file locations, and permissions.
3.  **Common Services:** Exploration and enumeration of **SSH, HTTP, MySQL, SMB, and FTP**.
4.  **Tools:** **Nmap** mastery (essential) and **Metasploit Framework** (auxiliary, but not mandatory).
5.  **Techniques:** Simple privilege escalation, **Pivoting** (movement between networks), and, mainly, **Enumeration**.

> **Golden Tip:** Enumeration is the most critical phase. If you are stuck, enumerate again. Most mistakes and those moments where we don't know what to do happen by ignoring a service or an open port.

### Recommendations and Methodology

* **Open Research:** You can consult documentation and Google during the exam. What is prohibited is external collaboration (or more famously, cheating).
* **Time Management:** With 48 hours, there is no need to hurry. If you feel tired or have a mental block, get away from the computer and come back later.
* **Dictionary Traps:** Avoid using massive wordlists like `rockyou.txt` on slow connections, unless strictly necessary. The exam VM already provides optimized lists for the scenario.

### Practice Labs (CTFs)

The machines below are excellent for solidifying the concepts required in the eJPT. Some may require a VIP subscription on the platforms by the time you are reading this (but it is worth it):

| Platform | Machine / Lab |
| :--- | :--- |
| **TryHackMe** | Ignite, RootMe, SimpleCTF, Startup, Blog, Anonymous, Internal, Poster, Source |
| **HackTheBox** | Oopsie, Pennyworth, Devel, Legacy, Lame, Love, Jeeves, Hawk |
---
If you can solve the machines above without constant aid from writeups, the exam will not present great technical difficulties. The biggest challenge is usually the psychological pressure and attention to detail during the information gathering phase.


### Long story short

> The **eJPT** was a great exam and experience for the start in this area that can be so deep. If you have the means to take it, I highly recommend it, even more so if you can catch it on **sale**, which makes the cost-benefit even better. It offers an **excellent base** for you to start your journey in the world of **offensive security**.