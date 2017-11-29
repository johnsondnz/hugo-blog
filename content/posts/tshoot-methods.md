---
title: "Troubleshooting Methods"
date: 2017-11-29T14:47:19+13:00
draft: true
categories: [ cisco, troubleshooting ]
tags: [ study-notes, tshoot, methodologies ]
---

## Troubleshooting Methods

### Top down
* Start at layer-7.
* Eliminates the most layers.
* Requires access to the application.

### Bottom Up
* Start at layer-1
* Requires access to the physical network.
* Requires detailed analysis of layer-1 and layer-2.
* Slowest.

### Divide and conquer
* Start at layer-3.
* Use ping and traceroute to test layer-3 reachability.
* Validate routing is as expected.
* Changes are if routing protocols are configured correctly it is a layer-1 or layer-2 issue.
* Firewalls might also be blocking traffic giving the false illusion that lower layers are at fault.

### Follow the path
* Traceroute and tcptrace.
* Ping hop by hop.

### Compare or Spot the difference
* Compare configurations.
* Look for odities.

### Component swapping or Move the problem
* Eliminate faulty equipment.
* Swap device/cabling, does the fault move?
* If the problem moves or goes away the device was at fault.

## 