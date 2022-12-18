---
layout: default
title: Lateral Movement
parent: Forensics
grand_parent: Security Bible
nav_order: 2
---

# Lateral Movement
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
### Remote Desktop (RDP)

#### source
``` yaml
Event Logs:
4648 - Logon specifying alt credentials
1024 - Dest Hostname
1102 - Dest IP address
Registry:
Remote desktop dest tracked per user:
NTUSER\Software\Microsoft\Terminal Server Client\Servers
ShimCache:
SYSTEM: mstsc.exe - remote desktop client
BAM/DAM:
SYSTEM: last time executed - mstsc.exe - last time executed
AmCache.hve:
First time executed: mstsc.exe
UserAssist:
NTUSER.DAT: mstsc.exe - remote desktop client execution
Last time executed 
Number of times executed
RecentApps:
NTUSER.DAT
Mstsc.exe - remote desktop client
Last time executed
Number of times executed
RecentItems subkey tracks connections dest and times
File system:
Jumplists: 
c:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\{MSTSC-APPID}-automaticdestinations-ms 
Tracks remote desktop connection dest and times
Prefetch:
c:\Windows\Prefetch\*mstsc.exe-{hash}.pf
Bitmap Cach:
c:\Users\<user>\AppData\Local\Microsoft\Terminal\Server Client\Cache
bcache##.bmc
cache####.bin
```

### Map Network Shares

#### source
``` yaml
Event Logs:
4648:  Logon specifying alt credentials
31001: Failed logon to dest

```
---