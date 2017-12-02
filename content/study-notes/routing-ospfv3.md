---
title: "OSPFv3"
date: 2017-11-30T15:31:19+13:00
draft: false
categories: [ study-notes ]
tags: [ ospf, ipv6, routing ]
highlight: false
---

## Basics
* Borrows many of [OSPFv2](/study-notes/routing-ospf.md) mechinisms.
* Interfaces are aded at the interface level with `ipv6 ospf <process> area <area>`.  There is no option under the router process.
  * The router will automatically create `ipv6 router ospf <process>`.  This cannot be deleted.  Doing so will stop the OSPFv3 process.
* Still uses an IPv4 RID.
  * uses `router-id` route process command if there is no IPv4 address configured on an interface.
* Unlike OSPFv2 you don't need to initiate OSPF with `router ospf` command.

## Differences from OSPFv2
* Uses `FF02::5` and `FF02::6` muticast addresses instead of `224.0.0.5` and `224.0.0.6`.
* Allows a link to be in one or more OSPF instances.
* Uses `ipv6` to prefix commands.
* `subnets` is no longer required when redistributing.

## Configure
### Assign router-id
For when there are no IPv4 addresses configure on any interfaces.  I also like to do this manually anyway.

```
router(config)# ipv6 router ospf <proces>
router(config-rtr)# router-id <address>
```

### Add interface to area
```
router(config)# interface <iface>
router(config-if)# ipv6 ospf <process> area <area>
```

## Verify
### Process
```
R4#show ipv6 ospf
 Routing Process "ospfv3 6" with ID 10.1.4.9
 Supports NSSA (compatible with RFC 3101)
 Supports Database Exchange Summary List Optimization (RFC 5243)
 Event-log enabled, Maximum number of events: 1000, Mode: cyclic
 It is an autonomous system boundary router
 Redistributing External Routes from,
    rip RIPng with metric-type 1
 Router is not originating router-LSAs with maximum metric
 Initial SPF schedule delay 5000 msecs
 Minimum hold time between two consecutive SPFs 10000 msecs
 Maximum wait time between two consecutive SPFs 10000 msecs
 Minimum LSA interval 5 secs
 Minimum LSA arrival 1000 msecs
 LSA group pacing timer 240 secs
 Interface flood pacing timer 33 msecs
 Retransmission pacing timer 66 msecs
 Retransmission limit dc 24 non-dc 24
 Number of external LSA 1. Checksum Sum 0x0050CB
 Number of areas in this router is 1. 1 normal 0 stub 0 nssa
 Graceful restart helper support enabled
 Reference bandwidth unit is 100 mbps
 RFC1583 compatibility enabled
    Area 34
        Number of interfaces in this area is 1
	SPF algorithm executed 14 times
	Number of LSA 8. Checksum Sum 0x03F2E5
	Number of DCbitless LSA 0
	Number of indication LSA 0
	Number of DoNotAge LSA 0
	Flood list length 0
```

### Interfaces
```
R4#show ipv6 ospf interface 
Tunnel34 is up, line protocol is up 
  Link Local Address FE80::A01:10A, Interface ID 14
  Area 34, Process ID 6, Instance ID 0, Router ID 10.1.4.9
  Network Type POINT_TO_POINT, Cost: 1000
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:07
  Graceful restart helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 10.1.1.9
  Suppress hello for 0 neighbor(s)
```

### Neighbours
```
R4#show ipv6 ospf neighbor 

            OSPFv3 Router with ID (10.1.4.9) (Process ID 6)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
10.1.1.9          0   FULL/  -        00:00:38    14              Tunnel34
```

### Routes
```
R4#show ipv6 route ospf
IPv6 Routing Table - default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
OI  2026::1:0/122 [110/1064]
     via FE80::A01:109, Tunnel34
OI  2026::12:0/122 [110/1128]
     via FE80::A01:109, Tunnel34
```

### Database
```
R4#show ipv6 ospf database 

            OSPFv3 Router with ID (10.1.4.9) (Process ID 6)

		Router Link States (Area 34)

ADV Router       Age         Seq#        Fragment ID  Link count  Bits
 10.1.1.9        411         0x80000009  0            1           B
 10.1.4.9        420         0x80000008  0            1           E

		Inter Area Prefix Link States (Area 34)

ADV Router       Age         Seq#        Prefix
 10.1.1.9        405         0x80000001  2026::1:0/122
 10.1.1.9        405         0x80000001  2026::12:0/122

		Link (Type-8) Link States (Area 34)

ADV Router       Age         Seq#        Link ID    Interface
 10.1.1.9        417         0x80000005  14         Tu34
 10.1.4.9        510         0x80000005  14         Tu34

		Intra Area Prefix Link States (Area 34)

ADV Router       Age         Seq#        Link ID    Ref-lstype  Ref-LSID
 10.1.1.9        1132        0x80000001  0          0x2001      0
 10.1.4.9        1122        0x80000001  0          0x2001      0

		Type-5 AS External Link States

ADV Router       Age         Seq#        Prefix
 10.1.4.9        509         0x80000001  2026::3:0/122
```
