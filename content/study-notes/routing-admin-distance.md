---
title: "Routing Protocol Admin Distance"
date: 2017-11-29T22:18:19+13:00
draft: false
categories: [ study-notes ]
highlight: false
tags: [ remind-me, junos, routing ]
highlight: false
---

__Cause I get confused between Junos and IOS!!__

## Cisco Protocol Admin Distances
* 0: Connected
* 1: Static
* 5: EIGRP summary route
* 20: eBGP
* 90: EIGRP (internal)
* 100: IGRP
* 110: OSPF
* 115: IS-IS
* 120: RIP
* 170: EIGRP (external)
* 200: iBGP

## Junos Route Preferences
* 0: Directly connected network
* 4: System routes
* 5: Static and Static LSPs
* 6: Static LSPs
* 7: RSVP-signaled LSPs 
* 9: LDP-signaled LSPs
* 10: OSPF internal route
* 15: IS-IS Level 1 internal route
* 18: IS-IS Level 2 internal route
* 30 Redirects
* 40: Kernel
* 50: SNMP
* 55: Router discovery
* 100: RIP
* 100: RIPng
* 105: PIM
* 110: DVMRP
* 130: Aggregate
* 150: OSPF AS external routes
* 160: IS-IS Level 1 external route
* 165: IS-IS Level 2 external route
* 170: BGP
* 175: MSDP
