---
title: "Loop and Root Guard"
date: 2017-11-28T16:18:19+13:00
draft: false
categories: [ cisco, switching ]
tags: [ spanning-tree, loop-prevention ]
---

## Short Version
* Loop Guard on non-designated ports, except edge ports.
* Root Guard on all designated ports.
* BPDU Guard and portfast on all edge ports.

## Loop Guard vs Root Guard

### Loop Guard
* Should be applied on **all non-designated ports**.
* Can be configured globally and per interface.
* Prevents an interface from transitioning to designated in the absence of BPDUs.
* Is exclusive from root guard.  The two cannot cannot be active at the same time.
* Ports that are forwarding and were receiving BPDUs and those BPDUs stop arrives are transitioned to **blocking**, **loop-inconsistent** state.
  * This happens prior to any **alternative** ports being transitioned to forwarding.
* Works very well with UDLD which can better detect shared link issues where loop guard cannot.
  * UDLD will also supports detection of unidirectional links on boot.. 
* When applied globally the switch only ports considered to be point-to-point links are enabled.
* Ports auto recover once BPDUs start being received again.

#### Configure per interface
```
interface <iface>
 spanning-tree guard loop
```

#### Configure globally
`switch(config)# spanning-tree gaurd loop default`


## Root Guard
* Configured on all **designated ports** except edge ports where BPDU Guard should be used.
* Prevents a normally designated port from transitioning to root.
  * Forces a port to become **designated** to prevent downstream switches from becoming the root bridge.
* Prevent surrounding switches from becoming root.
* Places a port into **listening**, **root-inconsistent** state on the detection of a superior BPDU.
* Ports auto recover once the superior BPDUs stop being received.

### Configure per interface
```
switch(config)# interface <iface>
switch(config-if)# spanning-tree guard root
```
