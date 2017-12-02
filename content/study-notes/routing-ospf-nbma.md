---
title: "OSPF Nonbroadcast Multiacces (NBMA)"
date: 2017-11-29T20:08:19+13:00
draft: false
categories: [ study-notes ]
tags: [ ospf, routing ] 
highlight: false ]
---

## NBMA Networks
* Typically applies to frame-relay networks.
* These networks don't have a multidestination capability, i.e. multicast or broadcast.

### RFC 2328:
  * Nonbroadcast: OSPF similates broadcast network.  Neighbours are manually configured.
    * One IP subnet.
    * Neighbours and manually configured.
    * DR and BDR elected, each needs full connectivity others.
    * Full or partial-mesh topology.
  * Point-to-multipoint: Used in partial mesh.  Treated as a collection of point-to-point links.  No DR and BDR is elected and neighbours are dynamically discovered.
    * One IP subnet.
    * Automatic OSPF neighbour discovery.
    * DR and BDR not required.
    * Typically used in a partial-mesh topology.

### Cisco special sauce
* Point-to-multipoint nonbroadcast:
 * Used as a replacement for RFC-compliant p2mp that has not broadcast capability.
 * Neighbors must be manually configured.
 * DR and BDR not required.
* Broadcast:
  * Makes the WAN appear as a LAN.
  * One IP subnet.
  * Automatic OSPF neighbour discovery.
  * DR and BDR elected.
  * Full and partial-mesh topology.
* Point-to-point
  * IP subnet per-link.
  * No DR and BDR elections.
  * Used only when two routers form an adjacency on a pair of interfaces.
  * Interfaces can be either WAN or LAN.

### Defaults
* OSPF type on a frame-relay sub-interface is point-to-point.
* OSPF type on a frame-relay multipoint sub-interface is nonbroadcast.
* OSPF mode on main frame-relay interface is also nonbroadcast.
