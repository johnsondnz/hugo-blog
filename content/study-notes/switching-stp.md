---
title: "Spanning-Tree"
date: 2017-11-29T15:14:19+13:00
draft: false
categories: [ cisco, switching ]
tags: [ spanning-tree ]
---

## Todo
* Split this into respective STP docs.
* Tidy this up, alot

## CST, PVST+, MST
* __CST__: Common spanning-tree.  All VLANs in the same instance.  One switch handles all traffic.
* __PVST+__: Per-VLAN spanning-tree plus.  SPT run per-vlan, very granular.
* __MST__: Multiple spanning-tree.  VLANs are mapped to one of 16 instance that have a common topology.
  * IST works with CST and is instance 0.  Can represent the entire MST region as a virtual CST bridge.
  * MSTI is an instance with VLAN mappings and is instance 1-15.
  * MSTI regions:
    * `name`, `revision` and `vlan-mapping table` must match for switches to be in the same region.

## Cisco MAC address
As Cisco uses per-vlan spanning-tree (PVST+) and as a result uses non-IEEE destination MAC for BPDUs.

* __Cisco__: 0100.0ccc.cccd for PVST and RSTP+.  Uses a 802.3 / 802.2 SNAP Frame.
* __IEEE-802.1d and 802.1w__: 0180.c200.0000 802.3 / 802.2 LLC Frame.
* __IEEE-802.1s__: ------------ DUNNO ------------

### Vendor interoperability
Cisco uses of non-IEEE standard protocols can result in other vendors not understanding the BPDU.  Some will drop the frame, others will switch it as a standard multicast frame.

Juniper has a Rapid-PVST+ equivilent called VSTP.  Cisco switches meed to use the `spanning-tree pathcost method long` command for interoperability, otherwise link costs will not align.

## STP (802.1d) vs RSTP (802.1w)

### Port States
* __STP Disabled__: RSTP Discarding
* __STP Blocking__: RSTP Discarding
* __STP Listening__: RSTP Discarding
* __STP Learning__: RSTP Learning
* __STP Forwarding__: RSTP Forwarding

### Timers
* __STP__: every 2 seconds by the root bridge.
* __RSTP__: every 2 seconds by every switch.

## Basics
* Lowest bridge-id wins root bridge election.
  * Bridge-id = bridge-priority (2 bytes) + MAC address (6 bytes)
* Cost is incremented on receipt of BPDUs not as they are sent.


## Root port selection
1. Lowest bridge-id.
2. Lowest path cost to the root bridge.
3. Lowest sender bridge-id.
4. Lowest port-priority.
5. Lowest port-id.


### Topology change notification (TCN)
* __STP__: Sent by the root bridge only out all its designated ports.  `max-age` + (2 x `forwarding delay`) required.
  * Only root can propogate a TCN Frame with TC bit set.  This means is can be very slow to popogate.
* __RSTP__: Sent by the detecting switch out all non-edge designated ports and root ports, resulting in much fast convergence.
  * TC bit set in ver2 BPDU frame.
  * Occurs as quickly as the TC propogation can occur.
  * Recieving switches flush mac-address table on all ports except the one which received the TCN.
  * Legacy switches are notified using the TCN BPDU as per 802.1d.

## Port costs
From [packetlife.net](http://packetlife.net/blog/2008/sep/5/spanning-tree-port-costs/).

### Short
![short](/img/8021d-1998-costs.png)

### Long
![long](/img/8021d-2004-costs.png)