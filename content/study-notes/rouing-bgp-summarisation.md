---
title: "BGP Summarisation"
date: 2017-11-30T21:41:19+13:00
draft: false
categories: [ study-notes ]
tags: [ bgp, summarisation, routing ]
highlight: false
---


## Summarisation
* BGP supports summarisation.


## Where and how
* In the route process `aggregate-address <address> <mask>`
* In the route process `aggregate-address <address> <mask> (<no-summary>|<uppress-map>) [supress-map-name]`

## What happens
* Without `summary-only` or `suppress-map` knobs both the aggregate and more-specific routes are advertised.
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
### BGP summarisation
```
router(config)# router bgp <as>
router(config-router)# aggregate-address <address> <mask> (<no-summary>|<uppress-map>) <[upress-map-name]`
```
