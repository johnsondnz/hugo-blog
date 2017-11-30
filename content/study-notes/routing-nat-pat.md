---
title: "NAT and PAT"
date: 2017-11-30T17:08:19+13:00
draft: false
categories: [ cisco ]
tags: [ nat ]
---

## Basics
* Translates RFC1918 to routed public addreses
* 1:1 burns a lot public addresses.  This is known as source nat (SNAT).
* Dynamic creates a pool on inside global address (DNAT).
* Port address translation overloads an interface so all users share a single inside global address (PAT).

### Address Types
* __Inside Local address__: RFC1918 address in my network.
* __Inside Global address__: Public IP address in my network.
* __Outside Local address__: RFC1918 address at a remote network.
* __Outside Global address__: Public IP address or a remote network.

#### It is all about perspective
* If the address is in use in your network (local or global) it is an inside address.
* If the address is external to your network it is an outside address.

### RFC1918 address space
* 10.0.0.0/8
* 172.16.0.0/212
* 192.168.0.0/16

## PAT
* Share an address for inside local to inside global translation.
* THe router randomises the source port number.
* ICMP traffic is given an ident number.

## Configure NAT

### Define interfaces
```
router(config)# interface <inside-iface>
router(config-if)# ip nat inside
!
router(config-if)# interface <outside-iface>
router(config-if)# ip nat outside
```

### Configure SNAT
```
router(config)# ip nat inside source static <address> <address>
```

### Configure DNAT
```
router(config)# access-list <1-99> permit <address> <mask>
router(config)# ip nat pool <name> <start-addres> <end-address> netmask <mask>
router(config)# ip nat inside source list <acl> pool <name>
```

or

```
router(config)# access-list <1-99> permit <address> <mask>
router(config)# ip nat pool <name> <start-addres> <end-address> prefix-length <0-32>
router(config)# ip nat inside source list <acl> pool <name>
```

### Configure PAT
```
router(config)# access-list <1-99> permit <address> <mask>
router(config)# ip nat inside source list <acl> interface <iface> overload
```

### Configure dynamic PAT
```
router(config)# access-list <1-99> permit <address> <mask>
router(config)# ip nat pool <name> <start-addres> <end-address> netmask <mask>
router(config)# ip nat inside source list <acl> pool <name> overload
```

or

```
router(config)# access-list <1-99> permit <address> <mask>
router(config)# ip nat pool <name> <start-addres> <end-address> prefix-length <0-32>
router(config)# ip nat inside source list <acl> pool <name> overload
```


## Verify
### Active translations
```
R1#show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.65.200.225:0  10.1.4.6:0         209.65.200.241:0   209.65.200.241:0
```

### Translation statistics
```
R1#show ip nat statistics 
Total active translations: 1 (0 static, 1 dynamic; 1 extended)
Peak translations: 2, occurred 01:16:12 ago
Outside interfaces:
  Serial1/1
Inside interfaces: 
  Serial1/0.112
Hits: 216  Misses: 0
CEF Translated packets: 14, CEF Punted packets: 0
Expired translations: 1
Dynamic mappings:
-- Inside Source
[Id: 1] access-list 1 interface Serial1/1 refcount 1

Total doors: 0
Appl doors: 0
Normal doors: 0
Queued Packets: 0
```