---
title: "Spanning Tree Uplinkfast"
date: 2017-11-28T23:50:19+13:00
draft: false
categories: [ study-notes ]
tags: [ spanning-tree, switching ]
highlight: false
---

## Uplinkfast
* Used to transition a **blocking** port into forwarding faster than the normal 50 second timer (max-age(20) + (2 x forwarding delay(15))
* Transition take around 1-3 seconds.
* Cisco best practice states only use in the access-layer.
* Enabled globally for all VLANs on the switch.  **All or nothing**.
* When the normal root port comes back up uplinkfast will transition back to that port.  Time this takes is 2 x forwarding delay + 5 seconds before the root port enters the forwarding state.

## Effects
* Bridge priority up to 49,152 making this switch less likely to be the root bridge for all VLANs.
* Port costs go up by 3000 making it unlikely that downstream switches will be less likely to use this switch as transit to the root.
* In some cases uplinkfast switches before MAC tables in other switches catch up.
  * `max-update-rate` is used to flood dummy multicast frames to 0100.0ccd.cdcd from a source of every MAC in the switches mac-address-table!!
  * Default update rate is 150pps.

## Configure globally
```
switch(config)# spanning-tree uplinkfast
```
