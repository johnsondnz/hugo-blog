---
title: "EIGRP Summarisation"
date: 2017-11-30T12:06:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ eigrp, summarisation ]
---


## Summarisation
* EIGRP support summarisation.


## Where and how
* On the interface using `ip summary-address eigrp <as> <address> <mask>`

## What happens
* Neighbour relationsips on the interface.
* Router where the summary-addres is created will create a summary address with a null0 destination with an AD of 5.
  * AD can be viewed with `show ip route <address> <mask>` command or the running-config in some cases.
* Downstream routers receive an EIGRP route with an AD of 90 like a normal internal EIGRP route.
* There must be a route with a longer prefix that lies within the summary-address prefix for the summary-address to be advertised.

## Tips
1. Convert the addres binary.
2. From left to right comapare the binary addresses.
3. When the bits no longer line up that is the summary address.
4. For the mask add a 1 for each common bit in the address and you have the mask.

### Example
Four addresses:

* `8.1.1.1/32` - __000010__00 00000001 00000001 00000001 mask 11111111
* `9.1.1.1/32` - __000010__01 00000001 00000001 00000001 mask 11111111
* `10.1.1.1/32` -__000010__10 00000001 00000001 00000001 mask 11111111
* `11.1.1.1/32` -__000010__11 00000001 00000001 00000001 mask 11111111
* Summary address is __000010__00 = 8.0.0.0.
* Mask is __111111__00 = 252.0.0.0.

## Configure
### ABSR summarisation
```
router(config)# interface <iface>
router(config-if)# ip summary-address eigrp <as> <address> <mask>
```

