---
title: "Spanning-Tree Portfast"
date: 2017-11-28T22:18:19+13:00
draft: false
categories: [ study-notes ]
tags: [ spanning-tree, switching ]
highlight: false
---


## Portfast
* Portfast enabled ports contine to send BPDUs every 2 seconds.  It does not turn off STP.
* When portfast is enabled globally with `spanning-tree portfast default` all access ports become portfast enabled.
  * Upon use of `switchport mode trunk` portfast is disabled and trunking mode continues.
  * Any port in `swithport mode dynamic desirable` mode that negotiates a trunk with DTP will also disable portfast.
* `switchport nonegotiate` turns off DTP and prevent dynamic trunks forming, portfast then stays active.
  * `switchport nonegoitate` and `switchport mode dynamic desirable` cannot be on the same interface.  They are mutually exclusive.
  * Both ends of a link must have DTP enabled and be in `swithport mode dynamic desirable` mode for a trunk to form, otherwise the port stays in access mode.
* Listening and learning states skipped saving 30 seconds.
* Can be enabled on access and trunk interfaces.
  * An warning message is displayed by IOS when enabling on a trunk advising to take care on ports leading to hub, bridges and switches.
* Can be enabled globally and on an interfaces.

## Configuration
### Globally
* When configuring globally a warning is displayed advising to disable explicitly on ports leading to hubs, bridges and switches.
  * The cli help states __enable portfast by default on all access ports__, whereas the warning states you need to disable on trunks.

```
spanning-tree portfast default
switch(config)# interface <iface>
switch(config-if)# no spanning-tree portfast
```

### Per interface
```
switch(config)# interface <iface>
switch(config-if)# spanning-tree portfast
```

## Verify
* Verify all trunks to be sure when using the global option. IOS shouldn't apply the configure to the trunk ports by default.

```
ASW1#show spanning-tree interface gi1/0 portfast 
VLAN0010            enabled

ASW1#show spanning-tree interface gi0/1 portfast 
VLAN0001            disabled
VLAN0010            disabled
VLAN0020            disabled
VLAN0200            disabled
```

# BPDU Guard
* `spanning-tree portfast` interface command is a prerequisite.
* Can be applied globally or per port.
* Should be applied to all **edge ports**.
* err-disables a port on detection of a BPDU.
* Recovery is not automated by default.
  * `shutdown`, `no shutdown` restores the interface but doesn't guarantee the absence of a BPDU.
  * `err-disable recovery` global command can be configured to attempt a restore every x seconds.
  * either option results in an *err-diabled* port in the presence of BPDUs.

### Configure per interface
```
switch(config)# interface <iface>
switch(config-if)# spanning-tree bpduguard enable
```

### Configure globally
```
switch(config)# spanning-tree portfast edge bpduguard default
```

or

```
switch(config)# spanning-tree portfast bpduguard default
```

## BPDU Filter
* Can be enabled on a `portfast` enabled port.

### Configure globally
```
switch(config)# spanning-tree bpdufilter default
```

### Configure per interface
```
switch(config)# interface <iface>
switch(config-if)# spanning-tree bpdufilter enable
```
