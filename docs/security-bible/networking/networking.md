---
layout: default
title: Networking
nav_order: 3
parent: Security Bible
permalink: /docs/security-bible/networking
has_children: true
---

# Networking
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## General Concepts 

### Autonomous System Number (ASN)

``` yaml
An autonomous system (AS) is a very large network or group of networks with a single routing policy. 
Each AS is assigned a unique ASN, which is a number that identifies the AS. An AS routing policy is 
a list of the IP address that the AS controls, plus a list of the other ASes to which it connects. 
The ASN (Autonomous System Number) are unique numbers that are only required for external 
communications with inter-network routers.

```

### Border Gateway Protocol (BGP)
``` yaml
ASes announce their routing policy to other ASers and routers vai the Border Gateway Protocol (BGP). 
BGP is the protocol for routing data packets between ASes. Without the routing information, 
operating the internet on a large scale would be impractical, as data packets would get lost or 
take too long. Each AS uses BGP to announce which IP address they are responsible for and which ASes
 the connect to. BGP routers take all the information from ASes around the world and put it into 
 databases called routing tables to determine fastest paths from AS to AS. When packets arrive, 
 BGP routers refer to their routing tables to determine which AS the packet should go to next.

With so many ASes in the world, BGP routers are constantly updating their routing tables. 
As networks go offline, new networks come online, and ASes expand or contract their IP address 
space, all of this information has to be announced via BGP so that BGP routers can adjust their 
routing tables.
```

### Demilitarized Zones (DMZ)
``` yaml
The term DMZ known as Demilitarized zone, is a type of network where you deploy all your public 
internet-facing servers. It could be a web server, mail server, FTP server, etc. It is similar to 
the LAN network, but there are some differences. While in the LAN network, most of the traffic will 
be outbound, meaning the users will initiate the traffic out to the internet and come back. In DMZ, 
however, most of the traffic will be inbound. You open a certain port, for example, http/HTTPS, 
for your web server, and anybody on the internet will be able to reach your web server inside the DMZ. 
We can also deploy a web server on the LAN side and allow external users to access the webserver from 
the internet, But that’s not a good security practice. If your web server on the LAN got compromised, 
your entire LAN side of the network would be affected. For any public-facing traffic, it is recommended 
to keep them on an isolated network such as DMZ instead on the LAN.
```

### Address Resolution Protocol (ARP)
``` yaml
ARP is a protocol specifically used to map IP network addresses to physical addresses, such as Ethernet addresses.

It translates 32-bits addresses to 48-bits addresses and vice versa. This is needed because the most common level of internet protocol(IP) we use today is 32-bits long and MAC addresses are 48-bits long.
```

### IPSec
``` yaml
secure network protocol suite that authenticates and encrypts packets of data to provide secure 
encrypted communication between two computers over an Internet Protocol network. It is used in 
virtual private networks (VPNs).
```
---