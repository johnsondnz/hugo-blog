---
title: "Spanning-Tree Backbonefast"
date: 2017-11-28T23:18:19+13:00
draft: false
categories: [ study-notes ]
tags: [ spanning-tree, switching ]
highlight: false
---


## Todo
* More reading. 
* Get better understanding.
## Backbonefast
* Cisco proprietary.
* Helps with recover from undirect link failure by detecting inferrior BPDU.

## Issue
### SW1 is the root
![stp-ok](/img/stp-ok.png)

### Link from SW to SW2 fails
* SW3 receives BPDUs from SW1 and SW2 each claiming they are root.
* SW3 ignores SW2 BPDUs and begins `max-age` timer.
* When `max-age` timer for SW2 expires the port transitions to the listening state.

![stp-fail](/img/stp-fail.png)

### STP recovers
* SW3 starts to relay SW1 BPDU to SW2.

![stp-recover](/img/stp-recover.png)

* STP is slow, 50 seconds to resolve issues.

## Backbone Fast Effects
* All or nothing for the backbone of the network due to RLQ messages.
* 20 second `max-age` timer is skipped.
* RLQ is used.  __Root Link Query__.
  * Act like a echo and echo-reply.

### Indirect Link failure detected
* RLQ is triggered and sent by the switch detecting the indirect link failure.
  * Query has the name of the switch that the switch believes is the root bridge.
  * If the returned response has the same name, everything is fine.  If not then hmmm.

## Configure
```
switch(config)# spanning-tree backbonefast
```

## Verify
```
ASW1#show spanning-tree backbonefast 
BackboneFast is disabled
```

__Note__: backbonefast and uplinkfast are built into 802.1w (RSTP).  There is no need to configure these in today's networks when using 802.1w, unless you are using traditional IEEE-802.1d STP.