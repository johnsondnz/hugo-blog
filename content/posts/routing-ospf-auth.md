---
title: "OSPF Authentication"
date: 2017-11-29T20:29:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ ospf ]
---

## Types
* Type 0: null (no authentication).
* Type 1: plain-text.
* Type 2: message-digest.

## Where
Can be applied on an entires area or a single network segment.

* Per-area.
* Per-segment.

When choosing per-area, all interfaces in the area must be configured with the apporpriate authentication key (md5 or plain-text).  The `router ospf` section mandates authentication for the entire area.  Unless you use the interface command `ip ospf authentication null`.

*__Note__*: It is possible to override the authentication-type at the interface level.  It is more specific configuration.

## Virtual Links
* Area authentication can and will break virtual links.
* The show commands a very cryptic.
* Basically if area 0 has authentication enabled the virtual-link __must__ too.

## Configure

### Interface plain-text
```
router(config)# interface <iface>
router(config-if)# ip ospf authentication
router(config-if)# ip ospf authentication-key <password>
```

### Interface message-digest
```
router(config)# interface <iface>
router(config-if)# ip ospf authentication message-digest
router(config-if)# ip ospf message-digest-key <1-255> md5 <password>
```

### Area plain-text
```
router(config-router)# area <area> authentication
router(config-router)# interface <iface>
router(config-if)# ip ospf authentication-key <password>
```

### Area message-digest
```
router(config-router)# area <area> authentication message-digest
router(config-router)# interface <iface>
router(config-if)# ip ospf message-digest-key <1-255> md5 <password>
```

### Area authentication with null on interface.
```
router(config-router)# area <area> authentication (<cr>|message-digest)
router(config-router)# interface <iface>
router(config-if)# ip ospf authentication null
```

## Virtual Links
### Virtual link plain-test authentication
```
router(config)# router ospf <process>
router(config-router)# area <transit-area> virtual-link <remote-rid> authentication-key <password>
```

__and on the isolated router as the virtual link is in area 0__.

```
router(config-router)# area 0 authentication
```

### Virtual link message-digest authentication
```
router(config)# router ospf <process>
router(config-router)# area <transit-area> virtual-link <remote-rid> message-digest-key <1-255> md5 <password>
```

__and on the isolated router as the virtual link is in area 0__.

```
router(config-router)# area 0 authentication message-digest
```

## Verify
[OSPF](/posts/routing-ospf/)
