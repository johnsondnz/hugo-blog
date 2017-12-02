---
title: "Policy Based Routing"
date: 2017-11-30T13:56:19+13:00
draft: false
categories: [ study-notes ]
tags: [ route-policy, routing ]
highlight: false
---

## Basics
* Applied on inbound to an interface only.
* Matches either source or source and destination of a packet.
* CEF since IOS version 12.0 natively suports PBR.  There is no special configuration required.
* Using source only results in 100% of traffic from that address following the policy.
* Deny statements don't drop traffic, they only deny access to the policy.
* To drop packets `set interface null0` needs to be used.
* Uses `route-map` and `ip policy route-map <name>` commands.
* Traffic is selected via access-lists.
* Does not work on locally generated traffic

## Local policy routing
* Required to impact route selection for router generating traffic.
* Applied on the local router to the source traffic.
* Applied globally with `ip local policy route-map <name>` command.


## Configure
### Source only
```
access-list <1-99> permit host <addres>
!
route-map <name<> permit <seq>
 match ip address <acl>
 set ip next-hop <addres> 
!
interface <iface>
 ip policy route-map <name>
```

### Source and destination
```
access-list <100-199> (permit|deny) ip <address> <mask> <address> <mask>
!
route-map <name<> permit <seq>
 match ip address <acl>
 set ip next-hop <addres> 
!
interface <iface>
 ip policy route-map <name>
```

### Local policy routing
```
access-list <100-199> (permit|deny) ip <address> <mask> <address> <mask>
!
route-map <name<> permit <seq>
 match ip address <acl>
 set ip next-hop <addres> 
!
ip local policy route-map <name>
```