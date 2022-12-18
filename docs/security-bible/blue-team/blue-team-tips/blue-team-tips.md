---
layout: default
title: Blue Team Tips
nav_order: 1
parent: Blue Team
permalink: /docs/security-bible/blue-team/blue-team-tips
grand_parent: Security Bible
---


# Blue Team Tips
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### Image File Execution Options (IFEO)
IFEOs can be set directly via the Registry or in Global Flags via the GFlags tool. 
IFEOs are represented as Debugger values in the Registry under HKLM\SOFTWARE{\Wow6432Node}\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\ 
where <executable> is the binary on which the debugger is attached.
IFEOs can also enable an arbitrary monitor program to be launched when a specified program silently exits

Canary tokens can be setup to Monitor Registry Key Image File Execution Options (IFEO) to find binary execution / suspicious commands without the binary even being present. https://canarytokens.org/

---