---
title: "IPv6"
date: 2017-11-30T14:04:19+13:00
draft: false
categories: [ study-notes ]
tags: [ ipv6, routed-protocols ]
highlight: false
---

## Basics
* 128-bit hexidecimal address.
* Eight sections of 4 bytes separated by seven colons.
* NAT no longer a thing!!
* IS-IS, OSPFv3 EIGRP, RIPng routing protocols
* Designed with summarisation in mind.
* Supports address compression.
* No broadcast.
* ICMPv6 used for layer-2 addres resolution.
* DHCPv6 or auto-configuration (SLAAC).
* Fragmentation is moved to an extension header.
* Intermediate routers not longer perform fragmentation.  If it is required the source router does this.
* Routers exchange routing information over link-local addresses and link-local multicast.

## Type of address
* Unicast.
  * Global unicast: 2000::/3
  * Link-local: fe80::/10
* Multicast: TRaffic is sent to every member of the group.
  * ff00::/8
* Anycast: Traffic is sent to the closest member of the group.
  1. Cloest connected neighbour.
  2. Routing protocol metric.
* Loopback.
  * ::1/128
* Unspecified address.
  * ::/128.
  * ::/0 is the IPv6 default route.

### Multicast addresses
* FF02::1 - All nodes on the local-link.
* FF02::2 - All routers on the local-link.
* FF02::5 - All OSPF routers.
* FF02::6 - All OSPF DRs.
* FF02::9 - All RIPng routers.
* FF02::10 - All EIGRP routers.
* FF02::1:FFzz:zzzz/104 - Solicited-node address.
  * Used is neighbour solicitation messages.
  * the z's = the least significant 24-bits of the unicast/address of the node.

### IPv4 compatible address.
* First 96-bits set to zero, ::x.x.x.x.
  * The remaining bits will be in hexidecimal.


### IP Header
* Version field: 6.
* Traffic class: was `TOS` in IPv4.  Handy for QoS.
* Flow label: Allows marking of packets to signify it as a particular flow, handy for QoS.
* Payload length: Same as `total length` in IPv4.
* Hop limit: Rought like `TTL` in IPv4.  Every hop decrements this, when it hits zero the packet is discards.
* Next header: Same as IPv4 `rotocol` field.
* Source/Destination address: Same, same.

### IPv4 fields that didn't cut the mustard
* Header length.
* Identification.
* Flags.
* Fragment offset.
* Header checksum.

## Compression options
* Leading zero, drops the leading zeros in each section.
  * Can be used an often as needed.
  * At least one character is required for each section.
* Uses two colons to compress zeros.  Can be used once per address.
* Combined leading zero and double colon compression.

### Examples
#### Leading zero
```
Original 1234:0000:1234:0000:1234:0000:1234:0123
Compressed 1234:0:1234:0:1234:0:1234:123
```

#### Full compression
```
Original 1111:0000:0000:1234:0011:0022:0033:0044
Compressed 1111::1234:11:22:33:44
```

## Interface Identifiers and EUI-64
* 64-bit value that is auto-assigned.
* RFC 2373.
* Interface uses own 48-bit MAC address to assign iteself an IPv6 address.
  * FFFE is dropped into the middle of the address to make it 64-bits long.

The 7th bit of the first octet is the universal / local bit, it:

* Describes whether the address is universally unique or locally unique.
* As a MAC address is considered universally unique the bit i set to 1.

### Example 1
1. Our MAC access is `00-01-02-aa-b-cc`
2. `FFFE` is inserted into the middle making it `00-01-02-FF-FE-aa-bb-cc`
3. The 7th bit is set to 1 creating a universally unique IPv6 address. `02-01-02-FF-FE-aa-bb-cc`
4. The 8th bit being used for mutlicast groups is not changed.
5. IPv6 link-local address `fe80::201:2ff:Efeaa:bbcc` is assigned.

```
Hex: 00-01-02-FF-FE-aa-bb-cc
Binary: 00000000 00000001 00000010 11111111 11111110 10101010 10111011 11001100
```
becomes
```
Binary: 00000010 00000001 00000010 11111111 11111110 10101010 10111011 11001100
Hex: 02-01-02-FF-FE-aa-bb-cc
```

### Example 2
1. Our MAC access is `00-0f-f7-c4-09-c0`
2. `FFFE` is inserted into the middle making it `00-0f-f7-ff-fe-c4-09-c0`
3. The 7th bit is set to 1 creating a universally unique IPv6 address. `02-0f-f7-ff-fe-c4-09-c0`
4. The 8th bit being used for mutlicast groups is not changed.
5. IPv6 link-local address `fe80::20f:f7ff:fec4:9c0` is assigned.

```
Hex: 00-0f-f7-ff-fe-c4-09-c0
Binary: 00000000 00001111 11110111 11111111 11111110 11000100 00001001 11000000
```
becomes
```
Binary: 00000010 00001111 11110111 11111111 11111110 11000100 00001001 11000000
Hex: 02-0f-f7-ff-fe-c4-09-c0
```

## IPV4 Autoconfiguration process
* Stateless or stateful.
* Stateful uses a DHCPv6 server.
* __S__tateless __A__ddress __A__uto__C__onfiguration (SLAAC) is not dependant on a server.  It uses EUI-64 to confiure its own link-local address.

### SLAAC Process
1. Device determines its EUI-64 link-local address.
2. Device does a Duplicate Address Detection (DAD) test using ICMPv6-ND.
   1. Sends a NS (neighbour solicitation) to `FF02::1:FF:zzzz`
   2. If no NA (neighbour advertisement) is received the device is good to go.
3. Device sends RS (router solicitation) to `FF02::2` (all routers).
  * Asks the routers for an RA (router advertisement).
  * Routers send these periodically anyway, but hosts a needy.
4. Router responds with an RA (router advertisement).
5. Host either gets IPv6 address from DHCPv6 server or uses EUI-64 to create a globally unique IPv6 addres for itself based on the IPv6 prefix in the RA.

#### Router advertisements
* Flags to indicate whether the host should use DHCPv6 server for addressing information.
* If DHCPv6 is in use the RA tells the host where the DHCPv6 server is.
* If no DHCPv6 server is used the RA contains the IPv6 global prefix and prefix lifetime information.

## Configure
### Enable IPv6
```
router(config)# ipv6 unicast-routing
router(config)# ipv6 cef
```

### Enable interface for IPv6
```
router(cofig)# interface <iface>
router(config-if)# ipv6 enable
```

### Configure RIPng
#### Start RIPng process and add interface
```
router(config)# ipv6 router rip <name>
router(config-rtr) interface <iface>
router(config-if)# ipv6 rip <name> enable 
``` 

#### Send default route
```
R4(config-if)#ipv6 rip RIPng default-information ?
  only       Advertise only the default route
  originate  Originate the default route
```

### Configure OSPFv3
[OSPFv3](/study-notes/routing-ospfv3)

### Configure EIGRP for IPv6
[EIGRP](/study-notes/routing-eigrp)
