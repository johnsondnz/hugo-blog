---
title: "SDM Templates"
date: 2017-11-29T16:19:19+13:00
draft: true
categories: [ cisco, switching ]
tags: [ templates ]
---

## Todo read more!!
[Cisco Site](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750x_3560x/software/release/12-2_53_se/configuration/guide/3750xscg/swsdm.html)

## What is it?
* Predefined templates for system resource (TCAM) allocation.
* Allows for easy switch resource allocation based on role.

### Access
If the MLS is running a lot of ACLs this template will allocate resources to handle the maximum number of ACLs.

```
switch(config)# sdm prefer access
```

### Default
Default. Treats all functions roughly the same.

```
switch(config)# sdm prefer default
```

### Dual IPv4 and IPv6
For dual-stack switches.  It doesn't support every IPv6 feature includng IPv6 multicast.  *__Do homework__*.

```
switch(config)# sdm prefer dual-ipv4-and-ipv6
```

### Routing
Enhances the environment for IPv4 unicast routing.

```
switch(config)# sdm prefer routing
```

### vlan
Supprts CAM growth to contain the maximum number of unicast MAC address.  *__This also disable hardware routing__*.

```
switch(config)# sdm prefer vlan
```

*__Note__*: Reload is required to change the allocation of system resources.

## Verify
```
switch# show sdm prefer 
  The current template is "desktop routing" template. 
  The selected template optimizes the resources in 
  the switch to support this level of features for 
  8 routed interfaces and 1024 VLANs. 

   number of unicast mac addresses:            3K 
   number of igmp groups + multicast routes:   1K 
   number of unicast routes:                   11K 
     number of directly connected hosts:       3K 
     number of indirect routes:                8K 
   number of qos aces:                         0.5K
  number of security aces:                     1K 

  On next reload, template will be "desktop vlan" template.
```