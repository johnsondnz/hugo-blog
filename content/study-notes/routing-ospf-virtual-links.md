---
title: "OSPF Virtual Links"
date: 2017-11-29T20:12:19+13:00
draft: false
categories: [ study-notes ]
tags: [ ospf, routing ]
highlight: false
---

## Why?
* To link isolated non-backbone areas to the backbone area, because all non-backbone areas must be connected to the backbone area (area 0 or area 0.0.0.0).
* Stitches OSPF together so it works.
* Is a bandaid!!.  They are not a solution for poor area design.
* In the configuration the command defines the transit area.
* The transit area cannot be any sort of stub area.
* You must you the RID of the remote router in the `area <area> virtual-link` route process command.
* Is a point-to-point link.

## Authenticaiton
[OSPF Authentication](/posts/routing-ospf-auth)

## Configure
Configured on the isolated router and the router on the border of area 0. For example.

R1 --- `area 0` --- R2 --- `area 1` --- R3 --- `area 34` --- R4

```
R3(config-router)# area 34 virtual-link <R4 RID>
R4(config-router)# area 34 virtual-link <R3 RID>
```

```
router(config)# router ospf <process>
router(config-router)# area <transit area> virtual-link <remote RID>
```

## Verify
```
router# show ip ospf virtual-link
Virtual Link OSPF_VL3 to router 1.1.1.1 is up
  Run as demand circuit
  DoNotAge LSA allowed.
  Transit area 1, via interface ATM2/0.20, Cost of using 65
  Transmit Delay is 1 sec, State POINT_TO_POINT,
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:01
    Adjacency State FULL (Hello suppressed)
    Index 1/2, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0) Next 0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec

```