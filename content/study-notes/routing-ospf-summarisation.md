---
title: "OSPF Summarisation"
date: 2017-11-30T08:23:56+13:00
draft: false
categories: [ study-notes ]
tags: [ ospf, summarisation, routing ]
highlight: false
---

## Summarisation
* By default type-3 and type-5 LSAs do not contain summarised routes.
* Inter-area summarisation occurs on the ABR.
* External route summarisation generally occurs on the ABSR.
* OSPF is classless which means that subnet information is carried in route advertisements.
* Reducess the route table through aggregation of routing prefixes.
* Relies on a well designed IP plan.
* With OSPF summarisation type-3 and type-5 LSAs can be summarised.

## Where and how
* ABR summarisation
  * `area <area> range <address> <mask>` command is used within the routing process.
  * The specified area is the area containing the routes to be summarised.
  * Routes appear in the desitnation area (generally backbone) as a type-3 LSA.
* ABSR summarisation.
  * `summary-address <address> <mask>` command is used within the routing process.

## What happens
* Downstream OSPF routers receive an internal OSPF route with a AD of 110.
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
### ABR summarisation
```
router(config)# router ospf <process>
router(config-router)# area <area> 8.0.0.0 252.0.0.0
```

### ABSR summarisation
```
router(config)# router ospf <process>
router(config-router)# summary-address 8.0.0.0 252.0.0.0
```

