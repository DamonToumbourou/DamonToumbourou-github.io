---
layout: default
title: Commandline Scripts
nav_order: 2
parent: Blue Team
permalink: /docs/security-bible/blue-team/commandline-scripts
grand_parent: Security Bible
---

# Commandline Scripts
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

#### Search for any value in AD
``` yaml
Get-ADUser -LDAPFilter “(arn=<search_term>)”
```
---