---
title: "EtherChannel"
date: 2017-11-29T11:20:19+13:00
draft: false
categories: [ cisco, switching ]
tags: [ interfaces, lacp, pagp, spanning-tree ]
---

## EtherChannel
* Logical bundle of 2-8 like type interfaces between two switches.
  * MLAG allows for multi-chasses etherchannel.
* Provides a single layer-2 or layer-3 logical interfaces to IOS.
* STP sees the logical interface as a single link.

## Protocols and modes
### LACP
  * IEEE-802.3ad, later moved into IEEE-802.1AX-2008.
  * Uses layer-2 multicast destiation of 0180.c200.0002.
    * This mac is shared by CDP, VTP, DTP, PAgP and UDLD.
  * __Active__: Turns on LACP unconditionally.
  * __Passive__: Enables LACP mode only if a LACP device is detected.  One port in a pair **must** be in Active. 
  * Supports the concept of hot-standby links, i.e. links 9+ are hot-standby.

### PAgP
  * Cisco Propietary
  * Uses layer-2 multicast destiation of 0100.0ccc.cccc.
  * __Desirable__: Turns on PAgp unconditionally.
  * __Auto__: Enables PAgP mode only if a PAgP device is detected.  One port in a pair **must** be in desirable. 

### On and Off
* __On__: Port is unconditionally in an etherchannel with no protocol.  Recommendation is to avoid this option and go with LACP as the IEEE standard.
* __Off__: Port is not in a bundle.

## Gotchas
* Ports in a bundle must agree on speed and duplex.
* Physical and Port-Channel ports must have the same native vlan.
* Physical and Port-Channel ports must have the same vlan list.
* Physical and Port-Channel ports must agree on interface-mode, i.e. access or trunk.
* Port-Secuirty enabled ports cannot be entered into a etherchannel bundle.
* Ports can be up and not bundled, meaning STP sees the physical ports not the logical ports.
* Each end of a etherchannel pair needs to run the same protocol.  Modes can differ within that protocol.
* You cannot change the protocol on a port without first removing the original configured protocol.
  * Issue `switch(config-if)# no channel-group <num>` first.

## Configure per interface
### On
```
switch(config)# interface <iface>
switch(config-if)# channel-group <num> mode on
```

### LACP
```
switch(config)# interface <iface>
switch(config-if)# channel-group <num> mode (active|passive)
```

### PAgP
```
switch(config)# interface <iface>
switch(config-if)# channel-group <num> mode (desirable|auto)
```

## Verify
```
DSW1#show etherchannel sum    
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 3
Number of aggregators:           3

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
12     Po12(SU)        LACP      Gi0/0(P)    Gi0/1(P)    
13     Po13(SU)         -        Gi1/0(P)    Gi1/1(P)    
14     Po14(SU)        PAgP      Gi0/2(P)    Gi0/3(P)    
```
